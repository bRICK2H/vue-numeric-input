<template>
	<input type="text"
		:value="initValue"
		@input="input($event)"
		@keydown="keydown($event)"
		@blur="blur($event)"
	>
</template>

<script>
	export default {
		name: 'cInput',
		props: {
			value: {
				type: [Number, String],
				default: 0
			},
			decimal: {
				type: [Number, String],
				default: 0
			}
		},
		data: () => ({
			initValue: null,
			prevValue: null,
			initDecimalLength: 0,
			dec: 0,
			selection: {},
			step: 0,
			isInteger: false,
			isNumeric: false,
			isDeletes: false,
			isBackspace: false,
			isSeparator: false,
		}),
		methods: {
			input(e) {
				const { target } = e

				// Если ставим число перед нулем и там больше ничего нет устанавливаем поз 1 или по дефолту
				this.step = /^0{1}\d{1}/.test(target.value) ? 1 : target.selectionStart

				// Parse and separate value
				target.value = this.setSeparatorSpace(this.parseValue(target.value))
				console.log(this.dec)
				const indexComma = this.prevValue.indexOf(',')
				const leftSide = target.value.length - this.dec - 1
				const side = this.step > leftSide ? 'right' : 'left'
				const isDiffValue = target.value.length !== this.prevValue.length
				const isSelectionBothSide = this.selection.s <= indexComma && this.selection.e > indexComma
				console.log({side}, this.selection, this.step, isSelectionBothSide, {indexComma})

				// Устанавливаем дефолтные шаги курсора
				target.setSelectionRange(this.step, this.step)

				// Далее пойдут нестрандартные условия для перемещения шагов курсора

				// При удалении остаток чисел в начале 3 и текущая длинна не совпадает с предыдущей уменьшаем шаг
				if (this.isBackspace && isDiffValue && /^\d{3}/.test(target.value)) {
					target.setSelectionRange(this.step - 1, this.step - 1)
				}

				// Если не дробное число и нажата точка или запятает, указываем таргет на запятую
				if (!this.isInteger && this.isSeparator) {
					const commaIndex = target.value.indexOf(',') + 1
					this.step = commaIndex
					target.setSelectionRange(this.step, this.step)
				}

				if (this.isDeletes) {
					if (this.isBackspace) {
						switch (side) {
							case 'left':
								console.log('backw/left', this.step)
								// При удалении крайнего левого числа устанавливаем курсор в 0-ю позицию
								if (!this.step) {
									target.setSelectionRange(0, 0)
								}
								break;
							case 'right':
								console.log('backw/right')
								// target.setSelectionRange()
								break;
						}
					}
				} else {
					switch (side) {
						case 'left':
							const posLeft = /^\d{1} \d+/.test(target.value) && this.selection.s === this.selection.e ? 1 : 0
							console.log('add/left', this.step + posLeft)
							target.setSelectionRange(this.step + posLeft, this.step + posLeft)
							break;
						case 'right':
							console.log('add/right')
							// target.setSelectionRange()
							break;
					}
				}
				
			},
			parseValue(val) {
				const sValue = String(val).replace(/ /g, '')

				if (this.isInteger) {
					return String(Number(sValue.replace(/[^\d]/g, '')))
				} else {
					console.log({sValue}, this.prevValue)
					let format =  sValue.replace(/[^\d\.,]/g, '').replace(/\./, ',')
					console.log('no int', {format})
					const countComma = format.match(/,/g)
					const unFormat = Number(format.replace(/,/, '.'))

					// это все лучше вынести в input func
					if (countComma && countComma.length > 1 || !countComma) {
						console.log('aaa', this.selection.s, this.selection.e)
						if (this.selection.s === this.selection.e) {
							return this.prevValue.replace(/ /g, '')
						} else {
							console.log({format})
							const formatToArr = format.split('')
							const posComma = this.isDeletes
								? this.prevValue[this.selection.s - 1] === ' ' ? -1 : 0
								: this.prevValue[this.selection.s - 1] === ' ' ? 0 : 1
							// фокус уходит ни туда из инпута нужно корректировать

							formatToArr.splice(this.selection.s + posComma, 0, '.')
							const fixedFormat = Number(formatToArr.join('')).toFixed(this.dec)
							return String(fixedFormat).replace(/\./, ',')
						}
					}
					// input func

					const fixedNumber = unFormat.toFixed(this.dec)
					console.log('no int', {fixedNumber}, countComma)


					if (this.dec >= this.initDecimalLength) {
						console.log('dec>', this.initDecimalLength)
						return String(fixedNumber).replace(/\./, ',')
					} else {
						const sValue = String(fixedNumber).replace(/\./, ',')
						const indexComma = sValue.indexOf(',')
						return `${sValue.slice(0, indexComma)}${sValue.slice(indexComma, (indexComma + 1) + this.dec)}`
					}
				}
			},
			setSeparatorSpace(value) {
				const newArrValue = []
				const arrValue = value.split('')
				const wholePart = arrValue.indexOf(',') !== -1
					? arrValue.splice(0, arrValue.indexOf(','))
					: arrValue.splice(0)

				for (let i = wholePart.length; i > 0; i -= 3) {
					i - 3 >= 0
						? newArrValue.unshift(' ', ...wholePart.splice([i] - 3, 3))
						: newArrValue.unshift(...wholePart)
				}
				newArrValue.push(...arrValue)
				return newArrValue.join('').trim()
			},
			keydown(e) {
				const { target } = e
				this.prevValue = target.value
				this.isDelete = e.code === 'Delete',
				this.isBackspace = e.code === 'Backspace'
				this.isDeletes = this.isDelete || this.isBackspace,
				this.isSeparator = ['Comma', 'Period'].includes(e.code),
				this.isNumeric = [0,1,2,3,4,5,6,7,8,9].map(curr => `Digit${curr}`).includes(e.code)
				this.selection = { s: target.selectionStart, e: target.selectionEnd }


				if (e.code === 'Escape') {
					console.log('esc')
					target.value = this.initValue
				}
			},
			blur(e) {
				const { target } = e
				this.initValue = target.value
			}

		},
		watch: {
			$props: {
				immediate: true,
				deep: true,
				handler(props) {
					const { decimal, value } = props
					console.log({decimal, value})
					this.dec = Number(decimal)

					const val = typeof value !== 'number'
						? Number(value.replace(/,/, '.'))
						: value
					
					this.isInteger = this.dec
						? Number.isInteger(val)
						: Number.isInteger(Math.floor(val))
					
					if (!this.isInteger) {
						console.log('!isint')
						const sValue = String(val)
						this.initDecimalLength = sValue.slice(sValue.indexOf('.') + 1).length
					}
						
					this.initValue = this.dec
						? this.setSeparatorSpace(this.parseValue(val))
						: this.setSeparatorSpace(this.parseValue(Math.floor(val)))
				}
			},
		},
	}
</script>

<style lang="scss">
	input {
		padding: 10px;
	}
</style>