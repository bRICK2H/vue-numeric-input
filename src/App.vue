<template>
	<div class="b-price">
		<div class="input-box b-price__input-box"
			:style="setStyleInputBox"
			:class="[
				{ 'input-box__limit': isToggleWarning },
				{ 'input-box__focus': isFocus },
				target,
				setClassDisabledContainer
			]"
		>
			<transition name="button-update">
				<div v-show="updateButton"
					class="button-update"
					:style="setStyleButtonUpdate"
					@click="updateValue"
				>
					<span class="button-update-icon">
						<VUpdate />
					</span>
				</div>
			</transition>

			<span v-show="isCurrency && !discount"
				class="input-box__sign"
				v-html="getSign"
				:style="setStyleSign"
				@click="focusToInput"
			></span>

			<input class="input-box__price"
				type="text"
				:ref="inputRef"
				:style="setStyleInput"
				:value="iValue"
				:disabled="disabled"
				@input="input($event)"
				@keydown="keyAction($event)"
				@focus="focus($event)"
				@blur="unfocus($event)"
			>

			<div v-if="discount"
				class="discount-box"
				:style="{ width: `${height * 2}px` }"
			>
				<span class="discount-item"
					:class="{ 'discount-item--active': discountType === 'currency' }"
					:style="{
						width: `${height}px`,
						cursor: disabled ? 'no-drop' : 'pointer'
					}"
					@click="!disabled ? setDiscount('currency') : false"
				>
					<VPercent />
				</span>
				<span class="discount-item"
					:class="{ 'discount-item--active': discountType === 'percent' }"
					:style="{
						width: `${height}px`,
						cursor: disabled ? 'no-drop' : 'pointer'
					}"
					@click="!disabled ? setDiscount('percent') : false"
				>
					<VCurrency />
				</span>
			</div>
		</div>

	</div>
</template>

<script>
import VUpdate from './assets/img/svg/v-update'
import VPercent from './assets/img/svg/v-percent'
import VCurrency from './assets/img/svg/v-currency'

export default {
	name: 'inputNumeric',
	components: {
		VUpdate,
		VPercent,
		VCurrency
	},
	props: {
		value: {
			type: [String, Number],
			default: 0
		},
		separator: {
			type: Boolean,
			default: true
		},
		decimal: {
			type: [String, Number],
			default: 0
		},
		min: {
			type: [String, Number],
			default: 0
		},
		max: {
			type: [String, Number],
			default: 100000
		},
		isDefaultReset: {
			type: Boolean,
			default: true
		},
		disabled: {
			type: Boolean,
			default: false
		},
		focusSelected: {
			type: Boolean,
			default: true
		},
		focusOnOpen: {
			type: Boolean,
			default: false
		},
		toggleFocus: {
			type: Boolean,
			default: false
		},
		width: {
			type: [Number, String],
			default: 150
		},
		height: {
			type: [Number, String],
			default: 37
		},
		discount: {
			type: Boolean,
			default: false
		},
		updateButton: {
			type: Boolean,
			default: false
		},
		currency: {
			type: String,
			default: 'df'
		},
		target: {
			type: String,
			default: ''
		},
	},
	data: () => ({
		inputRef: '',
		iValue: 0,
		step: 0,
		prevValue: 0,
		prevStateValue: 0,
		iDecimal: 0,
		prevOffset: 0,
		selection: { s: 0, e: 0 },
		isSelectableValue: false,
		isInteger: false,
		isPressedPoint: false,
		isPressedComma: false,
		isDelimiter: false,
		isPressedMinus: false,
		isToggleMinus: false,
		isToggleWarning: false,
		isNumeric: false,
		isDeletes: false,
		isDelete: false,
		isBackspace: false,
		isLimit: false,
		isLoad: false,
		isFocus: false,
		isFirstUpdate: false,
		isWatch: true,
		currencies: {
			df: { sign: '', pos: 'right', order: 0 },
			en: { sign: '&#36;', pos: 'left', order: 1 },
			lb: { sign: '&#163;', pos: 'left', order: 1 },
			ru: { sign: '&#8381;', pos: 'right', order: 0 },
		},
		discountType: 'percent',
		discountAnimate: false,
		timeoutID: null
	}),
	computed: {
		isCurrency() {
			return !!this.currency && this.currency in this.currencies
		},
		getCurrency() {
			return this.isCurrency ? this.currencies[this.currency] : null
		},
		getSign() {
			return this.isCurrency ? this.getCurrency.sign : null
		},
		setStyleInput() {
			return {
				textAlign: this.isCurrency ? this.getCurrency.pos : 'right',
				[`padding-${this.isCurrency ? this.getCurrency.pos : 'right'}`]: '21px',
				cursor: this.disabled ? 'no-drop' : 'default'
			}
		},
		setStyleSign() {
			return { [this.isCurrency ? this.getCurrency.pos : 'right']: '0' }
		},
		setStyleInputBox() {
			return {
				height: !isNaN(Number(this.height))
					? `${this.height}px`
					: this.height,
				width: !isNaN(Number(this.width))
					? `${this.width}px`
					: this.width,
			}
		},
		setStyleButtonUpdate() {
			return {
				minWidth: `${this.height / 2}px`,
				height: `${this.height / 2}px`,
				order: this.isCurrency
					? this.getCurrency.order
					: 1
			}
		},
		setClassDisabledContainer() {
			return this.disabled ? 'input-box--disabled' : null
		},
		valueToNumber() {
			return Number(this.iValue.replace(/ /, '').replace(/,/, '.'))
		}
	},
	methods: {
		input(e) {
			const { target } = e
			
			this.isWatch = false
			this.isFirstUpdate = true
			this.step = target.selectionStart
			setTimeout(() => this.isWatch = true)

			const convertedValue = this.convertValue(target.value),
					numericValue = this.isInteger
						? convertedValue
						: Number(convertedValue.replace(/,/, '.')),
					totalInnerValue = this.definedNegativeOrPositiveValue(
						this.separator
							? this.separatorValue(convertedValue)
							: convertedValue,
						'string'
					),
					totalOuterValue = this.definedNegativeOrPositiveValue(numericValue, 'number'),
					isNegativeValue = /-/.test(totalInnerValue),
					isMaxPositive = !isNegativeValue && totalOuterValue > +this.max,
					isMaxNegative = isNegativeValue && totalOuterValue < +this.min

			this.isLimit = isMaxPositive || isMaxNegative

			if (this.isLimit) {
				this.isToggleWarning = true
				setTimeout(() => this.isToggleWarning = false, 300)

				if (!(/-/.test(this.prevValue))) this.isToggleMinus = false

				target.value = this.prevValue
				target.setSelectionRange(this.step - 1, this.step - 1)

			} else {
				target.value = this.iValue = totalInnerValue
				this.$emit('input', !this.discount
					? totalOuterValue
					: {
						value: totalOuterValue,
						type: this.discountType
					}
				)
				this.setCursorPosition(target)
			}
		},
		focus(e) {
			if (!this.focusSelected) return

			const { target } = e
			this.isFocus = true
			target.select()
		},
		unfocus(e) {
			const { target } = e
			this.isFocus = false
			this.prevStateValue = target.value
			this.$emit('blur')
		},
		focusToInput(e) {
			const { target } = e
			target.nextSibling.select()
		},
		definedNegativeOrPositiveValue(value, toType) {
			return toType === 'number'
				? this.isToggleMinus
					? value * -1 : value * 1
				: this.isToggleMinus 
					? `-${value}` : value
		},
		propsDefinition(props) {
			const { decimal, value } = props,
					iDecimal = +decimal,
					iValue = (() => {
						const toNumber = typeof value === 'string'
							? Number(value.replace(/,/, '.'))
							: value

						return toNumber.toFixed(iDecimal).replace(/\./, ',')
					})(),
					valueToNumber = Number(iValue.replace(/,/, '.'))

			this.iValue = this.separator ? this.separatorValue(iValue) : iValue
			this.iDecimal = +decimal
			this.isInteger = Number.isInteger(valueToNumber) && !iDecimal

			this.$emit('input', !this.discount
					? valueToNumber
					: {
						value: valueToNumber,
						type: this.discountType
					}
				)
		},
		dataDefinition() {
			this.prevOffset = this.iValue.length - this.iValue.replace(/ /g, '').length
			this.isToggleMinus = /-/.test(this.iValue)
			this.prevStateValue = this.iValue
			this.isLoad = true
		},
		convertValue(value) {
			const defineZero = 0,
					unSeparateValue = value.replace(/ /g, ''),
					currNumeric = Number(unSeparateValue.replace(/,/, '.')),
					isCurrNumeric = !isNaN(currNumeric),
					isNegativeSelectionValue = /-/.test(this.prevValue) && !(/-/.test(value))

					if (isNegativeSelectionValue) this.isToggleMinus = false

			if (this.isInteger) {
				const isOnlyMinus = unSeparateValue === '-'

				if (this.isPressedMinus || !isCurrNumeric) {
					if (/^-?[\.,]?$/.test(value)) {
						return defineZero.toFixed(this.iDecimal).replace(/\./, ',')
					}

					return isOnlyMinus ? 0 : this.prevValue.replace(/[^\d]/g, '')
				}

				if (isCurrNumeric) {
					const clearValue = unSeparateValue.replace(/[^\d]/g, '')
					return Number(clearValue)
				}
			} else {
				const isExistComma = /,/.test(value)

				if (!isExistComma) {
					if (!this.isSelectableValue) {
						return this.prevValue.replace(/ /g, '').replace(/-/, '')
					} else {
						if (!isCurrNumeric) {
							if (this.isPressedPoint || this.isDeletes) {
								return defineZero.toFixed(this.iDecimal).replace(/\./, ',')
							}

							return this.prevValue.replace(/ /g, '').replace(/-/, '')
						} else {
							const restValue = value.split('')
							const prevRestValue = this.prevValue.split('')
							let resultValue

							if(this.isDelimiter) {
								prevRestValue.splice(this.selection.s, this.selection.e - this.selection.s , '.')
								resultValue = prevRestValue
							} else {
								restValue.splice(this.step, 0 , '.')
								resultValue = restValue
							}

							const formatValue = resultValue.join('').replace(/ /g, '').split('.')
							const [left, right ] = formatValue

							if (!left) formatValue[0] = '0'
							if (!right) formatValue[1] = '0'

							return Number(formatValue.join('.')).toFixed(this.iDecimal).replace(/\./, ',').replace(/-/, '')
						}
					}
				} else {
					if (!this.isDelimiter && !this.isSelectableValue) {
						const clearValue = unSeparateValue.replace(/[^\d,]/g, '')
						const fixedValue = Number(clearValue.replace(/,/, '.')).toFixed(this.iDecimal)

						return fixedValue.replace(/\./, ',')
					}

					if (!this.isSelectableValue) {
						if (this.isDelimiter) {
							const arrValue = value.split('')
							arrValue.splice(this.step - 1, 1)

							return arrValue.join('').replace(/ /g, '').replace(/-/, '')
						}
					} else {
						if (!this.isDelimiter && !this.isPressedMinus && isCurrNumeric) {
							const clearValue = unSeparateValue.replace(/[^\d,]/g, '')
							const fixedValue = Number(clearValue.replace(/,/, '.')).toFixed(this.iDecimal)
			
							return fixedValue.replace(/\./, ',').replace(/-/, '')
						}

						if (!this.isSelectableValue) {
							if (this.isDelimiter) {
								const arrValue = value.split('')
								arrValue.splice(this.step - 1, 1)

								return arrValue.join('').replace(/ /g, '')
							}
						} else {
							if (this.isDelimiter) {
								const restValue = this.prevValue.split('')
								const res = restValue.splice(this.selection.s, this.selection.e - this.selection.s)

								if (res.includes(',')) {
									restValue.splice(this.selection.s, 0, '.')
								}

								const formatValue = restValue.join('').replace(/ /g, '').replace(/,/, '.').split('.')
								const [ left, right ] = formatValue

								if (!left) formatValue[0] = '0'
								if (!right) formatValue[1] = '0'
								
								return Number(formatValue.join('.')).toFixed(this.iDecimal).replace(/\./, ',').replace(/-/, '')
							} else {
								return this.prevValue.replace(/ /g, '').replace(/-/, '')
							}
						}
					}
				}
			}
		},
		separatorValue(value) {
			const newArrValue = [],
					arrValue = String(value).split(''),
					wholePart = arrValue.indexOf(',') !== -1
						? arrValue.splice(0, arrValue.indexOf(','))
						: arrValue.splice(0)

			for (let i = wholePart.length; i > 0; i -= 3) {
				const isSeparator = wholePart.length === 3 || wholePart.length === 4 && wholePart[0] === '-'

				i - 3 >= 0 && wholePart.length !== 3
					?  isSeparator
						? false
						: newArrValue.unshift(' ', ...wholePart.splice([i] - 3, 3))
					: newArrValue.unshift(...wholePart)
			}

			newArrValue.push(...arrValue)
			return newArrValue.join('')
		},
		setCursorPosition(target) {
			const currOffset = target.value.length - target.value.replace(/ /g, '').length,
					offsetLengthSpace = this.prevOffset === currOffset ? 0 : 1,
					leftSideIndex = target.value.length - this.iDecimal - (this.iDecimal ? 1 : 0),
					leftSide = target.value.slice(0, leftSideIndex),
					formatLeftSide = Number(leftSide.replace(/ /g, ''))
					
			this.prevOffset = currOffset

			const setSelectableStep = () => {
				const restValueAfterDel = this.prevValue.split('')
				restValueAfterDel.splice(this.selection.s, this.selection.e - this.selection.s)
				const prevLengthSpace = restValueAfterDel.join('').match(/ /g)
					? restValueAfterDel.join('').match(/ /g).length : 0
				const currLengthSpace = target.value.match(/ /g) ? target.value.match(/ /g).length : 0

				if (!this.step) {
					this.step = !formatLeftSide ? 1 : prevLengthSpace - currLengthSpace
				}

				if (prevLengthSpace > currLengthSpace) {
					this.step -= 1
				} else if (prevLengthSpace < currLengthSpace) {
					this.step += 1
				} else {
					if (this.step === 1 && /-/.test(leftSide)) {
						this.step = 2
					}
				}
			}


			if (this.isDeletes) {
				if (this.isSelectableValue) {
					setSelectableStep()
				}
				if (this.isBackspace) {
					if (!this.isSelectableValue) {
						if ((this.step - offsetLengthSpace) === 0 || (this.step - offsetLengthSpace) === -1) {
							if (this.step === 0 && formatLeftSide === 0) {
								this.step = /-/.test(this.prevValue) ? 0 : 1
							} else {
								this.step = 0
							}
						} else {
							this.step = this.step === 1 && formatLeftSide === -0
								? 2
								: this.step - offsetLengthSpace
						}
					}
				} else {
					if (!this.isSelectableValue) {
						const empty = ' '
						const prevLeftSide = this.prevValue.length - this.iDecimal - 1

						if (this.step && prevLeftSide > this.step && !(/^-\d{3} ?/.test(leftSide))) {
							if (leftSide === '-0') {
								this.step = 2
							} else {
								this.step = this.prevValue[this.step] === empty ? this.step + 1 : this.step - offsetLengthSpace
							}
						} else if (/^0/.test(target.value) && prevLeftSide > this.step) {
							this.step = 1
						} else if (this.step === prevLeftSide) {
							this.step += 1
						}
					}
				}
				
			} else {
				if (this.isNumeric) {
					if (this.isSelectableValue) {
						setSelectableStep()
					} else {
						if (!this.isInteger && leftSideIndex === 1 && this.step <= 2) {
							this.step = 1
						} else {
							if (this.step <= 3 && /-0,/.test(this.prevValue)) {
								this.step = 2
							} else {
								this.step = this.step + offsetLengthSpace
							}
						}
					}
				} else {
					if (this.isSelectableValue) {
						if (!this.isDelimiter && !this.isPressedMinus) {
							target.setSelectionRange(this.selection.s, this.selection.e)
							return
						} else {
							this.step = this.isPressedMinus
								? this.isToggleMinus ? 1 : 0
								: leftSideIndex + 1
						}
					} else {
						if (this.isDelimiter) {
							this.step = leftSideIndex + 1
						} else {
							this.step -= 1
						}
						if (this.isPressedMinus) {
							this.isToggleMinus
								? target.setSelectionRange(this.step + 1, this.step + 1)
								: target.setSelectionRange(this.step - 1, this.step - 1)
							return
						}
					}
				}

			}
			
			target.setSelectionRange(this.step, this.step)
		},
		keyAction(e) {
			const { target, code, key } = e
			
			this.prevValue = target.value
			this.selection.s = target.selectionStart
			this.selection.e = target.selectionEnd
			this.isSelectableValue = this.selection.s !== this.selection.e,
			this.isDelimiter = ['Comma', 'Period', 'NumpadDecimal'].includes(code)
			this.isPressedPoint = code === 'Period' && key === '.'
			this.isPressedComma = code === 'Comma' && key === ','
			this.isDelete = code === 'Delete'
			this.isBackspace = code === 'Backspace'
			this.isEscape = code === 'Escape'
			this.isDeletes = this.isDelete || this.isBackspace
			this.isNumeric = [0,1,2,3,4,5,6,7,8,9].some(curr => (code === `Digit${curr}` || code === `Numpad${curr}`))
			this.isPressedMinus = code === 'Minus'

			if (this.isPressedMinus) {
				this.isToggleMinus = !this.isToggleMinus
			}

			if (this.isEscape && this.isDefaultReset) {
				const valueToNumber = Number(this.prevStateValue.replace(/ /g, '').replace(/,/, '.'))
				this.isToggleMinus = /-/.test(this.prevStateValue)
				target.value = this.definedNegativeOrPositiveValue(this.prevStateValue.replace(/-/, ''))
				
				const totalValue = this.definedNegativeOrPositiveValue(valueToNumber, 'number')
				this.$emit('input', !this.discount
					? totalValue
					: {
						value: totalValue,
						type: this.discountType
					}
				)
				target.select()
			}

		},
		updateValue() {
			this.discountAnimate = true
			clearTimeout(this.timeoutID)
			this.timeoutID = setTimeout(() => this.discountAnimate = false, 300)

			this.$emit('update:value')
		},
		setDiscount(type) {
			this.discountType = type
			this.$emit('input', {
				type,
				value: this.valueToNumber
			})
		},
	},
	watch: {
		value: {
			immediate: true,
			handler() {
				this.propsDefinition(this.$props)
				if (!this.isLoad) this.dataDefinition()
			}
		},
		toggleFocus: {
			immediate: true,
			async handler(focus) {
				await this.$nextTick()
				focus
					? this.$refs[this.inputRef].focus()
					: this.$refs[this.inputRef].blur()
			}
		}
	},
	updated() {
		if (!this.isFirstUpdate) {
			this.dataDefinition()
			this.isFirstUpdate = true
		}
	},
	created() {
		this.inputRef = `input-ref:${String(Math.random()).slice(2, 10)}`
	},
	mounted() {
		if (this.focusOnOpen) {
			this.$refs[this.inputRef].focus()
		}
	},
	beforeDestroy() {
		this.$emit('beforeClose')
	}
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
		border: 2px solid #eeedf7;
    	border-radius: 8px;

		&__sign {
			height: 100%;
			width: 20px;
			display: flex;
			justify-content: center;
			align-items: center;
			font-family: 'Inter', sans-serif;
			font-size: 17px;
			color: #5f5f5f;
			position: absolute;
			user-select: none;
		}

		&__price {
			width: 100%;
			font-size: 14px;
			padding: 0;
			border: none;
			outline: none;
			background: none;
		}

		&__limit {
			animation: limit .2s ease;

			@keyframes limit {
				0% {
					left: 3px;
					background-color: rgba(255, 99, 71, 0.219);
					border: 2px solid rgba(255, 99, 71, 0.219);
				}
				25% {
					left: -2px;
					background-color: rgba(255, 99, 71, 0.219);
					border: 2px solid rgba(255, 99, 71, 0.219);
				}
				50% { left: 2px; }
				75% { left: -1px; }
			}
		}

		&__focus {
			border: 2px solid #cfcde5;
		}

		&--disabled { 
			opacity: .6;
		}
	}

	// discount
	.discount-box {
		height: 100%;
		display: flex;
		border-left: 2px solid #eeedf7;
	}
	.discount-item {
		display: flex;
		align-items: center;
		justify-content: center;
		background-color: #fff;
		transition: .2s;

		&:hover {
			background-color: rgba(238, 237, 247, .2);
		}

		&--active {
			background-color: #eeedf7;

			&:hover {
				background-color: rgba(238, 237, 247, .8);
			}
		}
		
		&:last-child {
			border-top-right-radius: 6px;
			border-bottom-right-radius: 6px;
		}
	}
	.button-update {
		margin: 0 5px;
		border-radius: 10px;
		outline: none;
		background-color: #fff;
		cursor: pointer;
		box-shadow: 0px 0px 4px rgba(26, 32, 44, 0.06), 0px 4px 10px -1px rgba(26, 32, 44, 0.06);

		&:hover {
			box-shadow: 0px 0px 4px rgba(26, 32, 44, 0.2), 0px 4px 10px -1px rgba(26, 32, 44, 0.2);
		}
	}
	.button-update-icon {
		width: 100%;
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
	}
	.button-update-enter-active {
		animation: update-enter .2s;

		@keyframes update-enter {
			0% { transform: scale(0); opacity: 0; }
		}
	}
	.button-update-leave-active {
		animation: update-leave .4s;

		@keyframes update-leave {
			50% { transform: rotate(180deg); }
			100% { opacity: 0; }
		}
	}
</style>