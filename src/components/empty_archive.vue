<template>
	<input type="text"
		:value="iValue"
		@input="input($event)"
		@keydown="keyAction($event)"
		@focus="focus($event)"
		@blur="unfocus($event)"
	>
</template>

<script>
export default {
	name: 'inputNumeric',
	props: {
		value: {
			type: [String, Number],
			default: 0,
		},
		decimal: {
			type: [String, Number],
			default: 0
		},
		separator: {
			type: Boolean,
			default: true
		},
	},
	data: () => ({
		step: 0,
		iValue: 0,
		prevValue: 0,
		prevStateValue: 0,
		iDecimal: 0,
		prevOffset: null,
		selection: { s: 0, e: 0 },
		isSelectableValue: false,
		isInteger: false,
		isPressedPoint: false,
		isPressedComma: false,
		isDelimiter: false,
		isPressedMinus: false,
		isToggleMinus: false,
		isNumeric: false,
		isDeletes: false,
		isDelete: false,
		isBackspace: false,
		isFocus: false,
		isLimit: false,
		isWatch: true,
		min: -Math.pow(10, 10),
		max: Math.pow(10, 10),
	}),
	methods: {
		input(e) {
			const { target } = e
			
			this.step = target.selectionStart
			this.isWatch = false
			setTimeout(() => this.isWatch = true)

			const convertedValue = this.convertValue(target.value),
					numericValue = this.isInteger
						? convertedValue
						: Number(convertedValue.replace(/,/, '.')),
					isNegativeValue = /-/.test(convertedValue),
					isMaxPositive = !isNegativeValue && numericValue > this.max,
					isMaxNegative = isNegativeValue && numericValue < this.min

			// В самом конце, когда все сделаю, по параметру isLimit подсвечивать input красным
			this.isLimit = isMaxPositive || isMaxNegative

			if (!this.isLimit) {
				target.value = this.definedNegativeOrPositiveValue(this.separator ? this.separatorValue(convertedValue) : convertedValue, 'string')
				this.$emit('input', this.definedNegativeOrPositiveValue(numericValue, 'number'))
				this.setCursorPosition(target)
			} else {
				target.value = this.prevValue
				target.setSelectionRange(this.step - 1, this.step - 1)
			}
		},
		focus(e) {
			const { target } = e

			this.isFocus = true
			target.select()
		},
		unfocus(e) {
			const { target } = e
			this.prevStateValue = target.value
		},
		dataDefinition(props) {
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

			this.$emit('input', valueToNumber)
		},
		definedNegativeOrPositiveValue(value, toType) {
			if (toType === 'number') {
				return this.isToggleMinus ? value * -1 : value * 1
			} else {
				return this.isToggleMinus ? `-${value}` : value
			}
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

						if (this.step && prevLeftSide > this.step) {
							this.step = this.prevValue[this.step] === empty ? this.step + 1 : this.step - offsetLengthSpace
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
			const { target } = e

			this.prevValue = target.value
			this.selection.s = target.selectionStart
			this.selection.e = target.selectionEnd
			this.isSelectableValue = this.selection.s !== this.selection.e,
			this.isDelimiter = ['Comma', 'Period'].includes(e.code)
			this.isPressedPoint = e.code === 'Period'
			this.isPressedComma = e.code === 'Comma'
			this.isDelete = e.code === 'Delete'
			this.isBackspace = e.code === 'Backspace'
			this.isEscape = e.code === 'Escape'
			this.isDeletes = this.isDelete || this.isBackspace
			this.isNumeric = [0,1,2,3,4,5,6,7,8,9].map(curr => `Digit${curr}`).includes(e.code)
			this.isPressedMinus = e.code === 'Minus'

			if (this.isPressedMinus) {
				this.isToggleMinus = !this.isToggleMinus
			}

			if (this.isEscape) {
				const valueToNumber = Number(this.prevStateValue.replace(/ /g, '').replace(/,/, '.'))
				this.isToggleMinus = /-/.test(this.prevStateValue)
				target.value = this.definedNegativeOrPositiveValue(this.prevStateValue.replace(/-/, ''))
				
				this.$emit('input', this.definedNegativeOrPositiveValue(valueToNumber, 'number'))
				target.select()
			}

		},
	},
	watch: {
		$props: {
			deep: true,
			handler(props) {
				if (this.isWatch) {
					this.dataDefinition(props)
				}
			}
		}
	},
	created() {
		this.dataDefinition(this.$props)
		
		this.prevOffset = this.iValue.length - this.iValue.replace(/ /g, '').length
		this.isToggleMinus = /-/.test(this.iValue)
		this.prevStateValue = this.iValue
	}
}
</script>

<style lang="scss">

</style>