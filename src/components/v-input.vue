<template>
	<div class="b-price">
		<div class="input-box b-price__input-box"
			:style="setStyleInputBox"
		>
			<span class="input-box__sign"
				v-html="getCurrency"
				:style="setStyleSign"
				@click="focusToInput($event)"
			></span>
			<input class="input-box__price"
				type="text"
				:style="setStyleInput"
				:value="initValue"
				@input="inputValue($event)"
				@focus="focus($event)"
				@focusout="unfocus"
				@keyup.esc="keyup($event)"
				@keydown="keydown($event)"
				@dblclick="isFocus = true"
				@click="clickCounter++"
			>
		</div>
	</div>
</template>

<script>
	export default {
		name: 'vInput',
		props: {
			value: {
				type: [String, Number],
				default: 0,
			},
			decimal: {
				type: Number,
				default: 0
			},
			width: {
				type: [Number, String],
				default: 100
			},
			height: {
				type: [Number, String],
				default: 37
			},
			currency: {
				type: String,
				default: 'ru'
			},
		},
		data: () => ({
			initValue: 0,
			prevValue: 0,
			beforeSelection: 0,
			clickCounter: 0,
			isFocus: false,
			isComma: false,
			isPoint: false,
			isDeleteForward: false,
			isBothDelete: false,
			isExhibitor: false,
			dedicated: [],
			dedicatedSelection: [],
			currencies: {
				df: { sign: '', pos: 'left' },
				en: { sign: '&#36;', pos: 'left' },
				lb: { sign: '&#163;', pos: 'left' },
				ru: { sign: '&#8381;', pos: 'right' },
			},
		}),
		computed: {
			getCurrency() {
				return this.currencies[this.currency].sign
			},
			setStyleInput() {
				return { textAlign: this.currencies[this.currency].pos,
				[`padding-${this.currencies[this.currency].pos}`]: '20px'
			}
			},
			setStyleSign() {
				return { [this.currencies[this.currency].pos]: '0' }
			},
			setStyleInputBox() {
				return {
					height: !isNaN(Number(this.height))
						? `${this.height}px`
						: this.height,
					width: !isNaN(Number(this.width))
						? `${this.width}px`
						: this.width
				}
			},
		},
		methods: {
			inputValue(e) {
				const { target } = e,
					isNumeric = !isNaN(Number(target.value))

				this.beforeSelection = target.selectionStart
				if (this.clickCounter >= 2) {
					this.isFocus = false
					this.clickCounter = 0
				}

				// Фокус
				if (this.isFocus) {
					if (!isNumeric) {
						target.value = this.parseValue(this.value)
						target.select()
						return
					}

					target.value = Number(target.value).toFixed(this.decimal)
					target.selectionStart = target.selectionEnd = 1
					this.isFocus = false
					this.$emit('input', Number(target.value))
					return
				}

				// Отсутствие дробной части
				if (!this.decimal) {
					target.value = Number(target.value.replace(/[^\d]/g, ''))
					this.$emit('input', Number(target.value))
					return
				}

				// Дробная часть присутствует
				if (!isNumeric || this.isExhibitor) {
					target.value = this.parseValue(this.value)
					target.setSelectionRange(...this.dedicatedSelection)

					if (this.isPoint || this.isComma) {
						target.value = Number(target.value).toFixed(this.decimal)
						target.selectionStart = target.selectionEnd = target.value.indexOf('.') + 1
					}

					return
				}

				const [before, after] = target.value.split('.'),
					beforeCommaLength = before.length + 1

				/**
				 * If cursor contains comma
				 */
				if (after === undefined) {
					const curArrayValue = !this.dedicated.length
						? String(this.value).split('')
						: String(this.parseValue(this.value)).split('')
					const [beforeIndex] = this.dedicatedSelection

					curArrayValue.forEach((curr, i) => {
						if (this.dedicated.includes(i) && curr.indexOf('.') === -1) {
							curArrayValue.splice(i, 1, '')
						}
					})

					if (e.data) curArrayValue.splice(beforeIndex, 1, e.data)
					if (isNaN(Number(curArrayValue.join('')))) {
						const zero = 0
						target.value = zero.toFixed(this.decimal)
					} else {
						target.value = Number(curArrayValue.join('')).toFixed(this.decimal)
					}

					target.selectionStart = target.selectionEnd = this.beforeSelection
					if (this.isDeleteForward) {
						target.selectionStart = target.selectionEnd = target.value.indexOf('.') + 1
					}
					this.$emit('input', Number(target.value))

					return
				}

				/**
				 * If cursor located in input
				 * left and right side
				 */
				if (target.selectionStart > before.length) {
					
					// right side if deleted value
					if (!this.isBothDelete) {
						const afterArr = after.split('')

						afterArr.splice(target.selectionStart - beforeCommaLength, 1)
						target.value = Number(`${before}.${afterArr.join('')}`).toFixed(this.decimal)
						this.$emit('input', Number(target.value))
						target.selectionStart = target.selectionEnd = this.beforeSelection

						const [, afterUp] = target.value.split('.')

						if (afterUp.length > this.decimal) {
							afterArr.splice(-1, 1)
							target.value = `${before}.${afterArr.join('')}`
							this.$emit('input', Number(target.value))
							target.selectionStart = target.selectionEnd = target.value.length
						}
					} else {
						// rigth side if added value
						const [, afterComma] = String(Number(target.value).toFixed(this.decimal)).split('.'),
							afterArr = afterComma.split(''),
							deletedIndex = afterArr.findIndex((el, i) => i === target.selectionStart - beforeCommaLength)

						target.value = Number(target.value).toFixed(this.decimal)
						this.$emit('input', Number(target.value))
						target.selectionStart = target.selectionEnd = beforeCommaLength + deletedIndex
					}

				} else {
					const isFirstZero = !Number(before[0])
					target.value = Number(target.value).toFixed(this.decimal)
					target.selectionStart = target.selectionEnd = this.beforeSelection

					if (isFirstZero && !this.isBothDelete) {
						target.selectionStart = target.selectionEnd = this.beforeSelection - 1
					}

					// left side if deleted lasted value
					if (before === '') {
						target.value = `${Number(before)}.${after}`
						target.selectionStart = target.selectionEnd = target.value.indexOf('.')
					}

					this.$emit('input', Number(target.value).toFixed(this.decimal))

				}

			},
			keydown(e) {
				const { target } = e,
					code = e.code,
					arrows = ['ArrowLeft', 'ArrowUp', 'ArrowRight', 'ArrowDown'],
					isComma = this.isComma = code === 'Comma',
					isPoint = this.isPoint = code === 'Period',
					isArrowsExist = arrows.includes(code)

				this.isExhibitor = code === 'KeyE'
				this.isDeleteForward = code === 'Delete'
				this.isBothDelete = ['Delete', 'Backspace'].includes(code)
				this.dedicatedSelection = [target.selectionStart, target.selectionEnd]
				this.dedicated = []
				for (let i = target.selectionStart; i < target.selectionEnd; i++) {
					this.dedicated.push(i)
				}

				if (isArrowsExist || isComma || isPoint) {
					this.isFocus = false
				}
			},
			keyup(e) {
				const { target } = e
				target.value = this.prevValue
				target.select()
				this.isFocus = true
				this.$emit('input', this.prevValue)
			},
			focus(e) {
				const { target } = e
				target.select()
				this.isFocus = true
			},
			focusToInput(e) {
				const { target } = e
				target.nextElementSibling.focus()
				this.isFocus = true
			},
			unfocus(e) {
				const { target } = e
				this.prevValue = target.value
				this.isFocus = false
			},
			parseValue(val) {
				let initValue = null
				const clearValue = String(val).replace(/[^\d\.]/g, '')
				const outerValue = (Number(Number(clearValue).toFixed(this.decimal)))

				if (!this.decimal) {
					initValue = Math.floor(outerValue)
				} else {
					const parseDecimal = String(outerValue).indexOf('.') !== -1
						? outerValue
						: outerValue.toFixed(this.decimal)
					const [, vAfterComma] = String(parseDecimal).split('.'),
						vAfterCommaLen = vAfterComma.length

					if (vAfterCommaLen > this.decimal) {
						const outerValueArr = String(outerValue).split(''),
							outerCommaIndex = String(outerValue).indexOf('.') + 1

						initValue = outerValueArr.splice(0, this.decimal + outerCommaIndex).join('')
					} else {
						initValue = outerValue.toFixed(this.decimal)
					}

				}

				return initValue
			}
		},
		created() {
			this.prevValue = this.initValue = this.parseValue(this.value)
		},
	}
</script>

<style lang="scss">
	.b-price {
		width: inherit;
		display: flex;
		align-items: center;

		&__input-box {
			display: flex;
			align-items: center;
		}
	}

	.input-box {
		width: auto;
		position: relative;
		border: 1px solid #cdcdcd;
		background: #fff;
		border-radius: 2px;

		&__sign {
			height: 100%;
			width: 20px;
			display: flex;
			justify-content: center;
			align-items: center;
			font-family: auto;
			font-size: 19px;
			color: #5f5f5f;
			position: absolute;
			user-select: none;
		}

		&__price {
			width: inherit;
			font-size: 14px;
			padding: 0;
			border: none;
			outline: none;
			background: none;
		}
	}
</style>