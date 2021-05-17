<template>
	<input type="text"
		:value="iValue"
		@input="input($event)"
		@keydown="keyAction($event)"
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
	},
	data: () => ({
		step: 0,
		iValue: 0,
		prevValue: 0,
		iDecimal: 0,
		prevOffset: null,
		selection: { s: 0, e: 0 },
		isSelectableValue: false,
		isInteger: false,
		isDelimiter: false,
		isPressedMinus: false,
		isToggleMinus: false,
		isNumeric: false,
		isDeletes: false,
		isDelete: false,
		isBackspace: false,
		isEvent: false,
		min: -Math.pow(10, 10),
		max: Math.pow(10, 10),
		isLimit: false,
	}),
	methods: {
		input(e) {
			const { target } = e
			
			this.isEvent = true
			setTimeout(() => this.isEvent = false)
			this.step = target.selectionStart

			const convertedValue = this.convertValue(target.value),
					numericValue = this.isInteger
						? convertedValue
						: Number(convertedValue.replace(/,/, '.')),
					isNegativeValue = /-/.test(convertedValue),
					isMaxPositive = !isNegativeValue && numericValue > this.max,
					isMaxNegative = isNegativeValue && numericValue < this.min
			console.log({numericValue})

			// В самом конце, когда все сделаю, по параметру isLimit подсвечивать input красным
			this.isLimit = isMaxPositive || isMaxNegative
			console.log('conver', this.separatorValue(convertedValue), convertedValue)

			if (!this.isLimit) {
				// console.log('tv', this.definedNegativeOrPositiveValue(this.separatorValue(convertedValue)))
				target.value = this.definedNegativeOrPositiveValue(this.separatorValue(convertedValue))
				this.$emit('input', this.definedNegativeOrPositiveValue(numericValue))
				this.setCursorPosition(target)
			} else {
				target.value = this.prevValue
				target.setSelectionRange(this.step - 1, this.step - 1)
			}
		},
		initialValue(value, iDecimal) {
			const toNumber = typeof value === 'string'
				? Number(value.replace(/,/, '.'))
				: value

			return toNumber.toFixed(iDecimal).replace(/\./, ',')
		},
		definedNegativeOrPositiveValue(value) {
			console.log({value}, this.isToggleMinus, 'П')

			if (typeof value === 'number') {
				// val = String(value).replace(/ /g, '').replace(/,/, '.').replace(/[^\d\.]/, '')
				console.log('number: ', this.isToggleMinus ? value * -1 : value * 1)
				return this.isToggleMinus ? value * -1 : value * 1
			} else {
				console.log('string', this.isToggleMinus ? `-${value}` : value)
				return this.isToggleMinus ? `-${value}` : value
			}
			
			/**
			 * По минусу
			 * 1. Сепаратор работает в некоторых моментах не правильно, создает лишний пробел
			 * 2. Так же при добовлении минуса съезжает курсор
			 * 3. При выделении значений числа, не должно ничего удаляться, должно сниматься выделение и тоглиться отр и полож знач.
			 * 4. Не удаляется минут через backspase
			 */
		},
		convertValue(value) {
			console.log(this.step, this.selection.s, value)
			
			const unSeparateValue = value.replace(/ /g, ''),
					currNumeric = Number(unSeparateValue.replace(/,/, '.')),
					isCurrNumeric = !isNaN(currNumeric)

			if (this.isInteger) {
				console.log('f:', unSeparateValue)
				const isOnlyMinus = unSeparateValue === '-',
						isDelMinus = !this.step && !(/-/.test(unSeparateValue)) && /-/.test(this.prevValue)
			
				if (isDelMinus) {
					this.isToggleMinus = false
				}

				if (this.isPressedMinus || !isCurrNumeric) {
					console.log(isOnlyMinus ? -0 : this.prevValue.replace(/[^\d]/g, ''))
					return isOnlyMinus ? 0 : this.prevValue.replace(/[^\d]/g, '')
				}

				if (isCurrNumeric) {
					console.log('1')
					const clearValue = unSeparateValue.replace(/[^\d]/g, '')
					return Number(clearValue)
				}
			} else {
				const isExistComma = /,/.test(value)

				if (!isExistComma) {
					if (!this.isSelectableValue) {
						return this.prevValue.replace(/ /g, '').replace(/-/, '')
					} else {
						console.log('foo')
						if (!isCurrNumeric) {
							return this.prevValue.replace(/ /g, '').replace(/-/, '')
						} 	else {
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
							const [ , right ] = formatValue

							if (!right) {
								formatValue[1] = '0'
							}

							console.log('to: ', Number(formatValue.join('.')).toFixed(this.iDecimal).replace(/\./, ',').replace(/-/, ''))
							
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
							if (!isCurrNumeric && !this.isDelimiter) {
								return this.prevValue.replace(/ /g, '').replace(/-/, '')
							}

							if (this.isDelimiter) {
								const restValue = this.prevValue.split('')
								let formatValue = ''
								
								if (!isExistComma) {
									restValue.splice(this.selection.s, this.selection.e - this.selection.s , '.')
									formatValue = restValue.join('').replace(/ /g, '').split('.')
								} else {
									restValue.splice(this.selection.s, this.selection.e - this.selection.s)
									formatValue = restValue.join('').replace(/ /g, '').replace(/,/, '.').split('.')
								}

								const [ , right ] = formatValue
	
								if (!right) {
									formatValue[1] = '0'
								}
									
								return Number(formatValue.join('.')).toFixed(this.iDecimal).replace(/\./, ',').replace(/-/, '')
							}

							if (this.isPressedMinus) {
								return this.isToggleMinus
									? this.prevValue.replace(/ /g, '')
									: this.prevValue.replace(/ /g, '').replace(/-/, '')
							}
						}
					}
				}
			}
		},
		separatorValue(value) {
			console.log({value}, 'В')
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

			console.log(newArrValue)
			newArrValue.push(...arrValue)
			return newArrValue.join('')
		},
		setCursorPosition(target) {
			const currOffset = target.value.length - target.value.replace(/ /g, '').length,
					offsetLengthSpace = this.prevOffset === currOffset ? 0 : 1,
					leftSideIndex = target.value.length - this.iDecimal - 1,
					leftSide = target.value.slice(0, leftSideIndex),
					formatLeftSide = Number(leftSide.replace(/ /g, ''))
					
			this.prevOffset = currOffset

			// const for selectable value (del/del+add)
	
			const setSelectableStep = () => {
				console.log('war')
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
				}
			}


			if (this.isDeletes) {
				if (this.isSelectableValue) {
					setSelectableStep()
				}
				if (this.isBackspace) {
					if (!this.isSelectableValue) {
						this.step = (this.step - offsetLengthSpace) === 0 || (this.step - offsetLengthSpace) === -1
							? this.step === 0 && formatLeftSide === 0 ? 1 : 0
							: this.step - offsetLengthSpace
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
						this.step = !this.isInteger && leftSideIndex === 1 && this.step <= 2
							? 1
							: this.step + offsetLengthSpace
					}
				} else {
					if (this.isSelectableValue) {
						if (!this.isDelimiter && !this.isPressedMinus) {
							target.setSelectionRange(this.selection.s, this.selection.e)
							return
						} else {
							console.log('here?')
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
							console.log('??')
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
			this.isDelete = e.code === 'Delete'
			this.isBackspace = e.code === 'Backspace'
			this.isDeletes = this.isDelete || this.isBackspace
			this.isNumeric = [0,1,2,3,4,5,6,7,8,9].map(curr => `Digit${curr}`).includes(e.code)
			this.isPressedMinus = e.code === 'Minus'

			if (this.isPressedMinus) {
				this.isToggleMinus = !this.isToggleMinus
			}
		},
	},
	watch: {
		$props: {
			deep: true,
			immediate: true,
			handler(props) {

				
				if (!this.isEvent) {
					console.log('this.isEvent', this.isEvent)
					const { decimal, value } = props,
							iDecimal = +decimal,
							iValue = this.initialValue(value, iDecimal),
							valueToNumber = Number(iValue.replace(/,/, '.'))
	
					this.iValue = this.separatorValue(iValue)
					this.iDecimal = iDecimal
					this.isInteger = Number.isInteger(valueToNumber) && !iDecimal
				}
				
			}
		}
	},
	created() {
		// this.iValue = this.definedNegativeOrPositiveValue(this.separatorValue(this.iValue))
		this.prevOffset = this.iValue.length - this.iValue.replace(/ /g, '').length
		this.isToggleMinus = /-/.test(this.iValue)
	}
}
</script>

<style lang="scss">

</style>