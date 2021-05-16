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
		}
	},
	data: () => ({
		step: 0,
		iValue: 0,
		prevValue: 0,
		iDecimal: 0,
		prevOffset: 0,
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
	}),
	methods: {
		input(e) {
			const { target } = e

			this.step = target.selectionStart

			const convertedValue = this.convertValue(target.value),
					numericValue = this.isInteger
						? convertedValue
						: Number(convertedValue.replace(/,/, '.')),
					isNegativeValue = /-/.test(convertedValue),
					isMaxPositive = !isNegativeValue && numericValue > Math.pow(10, 10),
					isMaxNegative = isNegativeValue && numericValue < -Math.pow(10, 10)

			if (isMaxPositive || isMaxNegative) {
				target.value = this.prevValue
				target.setSelectionRange(this.step - 1, this.step - 1)
				return
			}

			console.log('value before: ', `'${this.separatorValue(convertedValue)}'`)
			target.value = this.separatorValue(convertedValue)
			this.$emit('input', numericValue)

			this.setCursorPosition(target)
		},
		initialValue(value, iDecimal) {
			const toNumber = typeof value === 'string'
				? Number(value.replace(/,/, '.'))
				: value

			return toNumber.toFixed(iDecimal).replace(/\./, ',')
		},
		convertValue(value) {
			const definedNegativeOrPositiveValue = value => {
				let val = String(value).replace(/ /g, '').replace(/,/, '.').replace(/[^\d\.]/, '')
				val = this.isToggleMinus ? val * -1 : val * 1

				console.log({val})
				/**
				 * По минусу
				 * 1. Сепаратор работает в некоторых моментах не правильно, создает лишний пробел
				 * 2. Так же при добовлении минуса съезжает курсор
				 * 3. При выделении значений числа, не должно ничего удаляться, должно сниматься выделение и тоглиться отр и полож знач.
				 * 4. Не удаляется минут через backspase
				 */
				
				return Number(val).toFixed(this.iDecimal).replace(/\./, ',')
			}
			
			const unSeparateValue = value.replace(/ /g, '')

			if (this.isInteger) {
				const clearValue = unSeparateValue.replace(/[^\d]/g, '')
				return definedNegativeOrPositiveValue(Number(clearValue))
			} else {
				const isExistComma = /,/.test(value),
						currNumeric = Number(unSeparateValue.replace(/,/, '.')),
						isCurrNumeric = !isNaN(currNumeric)

				const	convertSelectableValue = value => {
					const spaceCount = value.match(/ /g) ? value.match(/ /g).length : 0,
							clearValue = String(Number(value.replace(/[^\d]/g, ''))),
							arrValue = clearValue.split('')

					arrValue.splice(this.isDelimiter ? this.step - 1 : this.step - spaceCount, 0, '.')
					const fixedValue = Number(arrValue.join('').replace(/ /g, '')).toFixed(this.iDecimal)

					return fixedValue.replace(/\./, ',')
				}


				if (!isExistComma) {
					if (!this.isSelectableValue) {
						return this.prevValue.replace(/ /g, '')
					} else {
						if (!isCurrNumeric) {
							return this.prevValue.replace(/ /g, '')
						} 	else {
							return convertSelectableValue(value)
						}
					}
				} else {
					if (!this.isDelimiter) {
						const clearValue = unSeparateValue.replace(/[^\d,]/g, '')
						const fixedValue = Number(clearValue.replace(/,/, '.')).toFixed(this.iDecimal)
	
						return fixedValue.replace(/\./, ',')
					}

					if (!this.isSelectableValue) {
						if (this.isDelimiter) {
							const arrValue = value.split('')
							arrValue.splice(this.step - 1, 1)

							return arrValue.join('').replace(/ /g, '')
						}
					} else {
						if (!isCurrNumeric && !this.isDelimiter) {
							return this.prevValue.replace(/ /g, '')
						}

						if (this.isDelimiter) {
							return convertSelectableValue(value)
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
				i - 3 >= 0
					? newArrValue.unshift(' ', ...wholePart.splice([i] - 3, 3))
					: newArrValue.unshift(...wholePart)
			}
			newArrValue.push(...arrValue)
			return newArrValue.join('').trim()
		},
		setCursorPosition(target) {
			const currOffset = target.value.length - target.value.replace(/ /g, '').length,
					offsetLengthSpace = this.prevOffset === currOffset ? 0 : 1,
					leftSideIndex = target.value.length - this.iDecimal - 1,
					leftSide = target.value.slice(0, leftSideIndex),
					formatLeftSide = Number(leftSide.replace(/ /g, ''))
					
			this.prevOffset = currOffset

			// const for selectable value (del/del+add)
			const restValueAfterDel = this.prevValue.split('')
			restValueAfterDel.splice(this.selection.s, this.selection.e)
			const prevLengthSpace = restValueAfterDel.join('').match(/ /g) ? restValueAfterDel.join('').match(/ /g).length : 0
			const currLengthSpace = target.value.match(/ /g) ? target.value.match(/ /g).length : 0


			if (this.isDeletes) {
				if (this.isSelectableValue) {
					if (!this.step) this.step = 1
					if (prevLengthSpace > currLengthSpace) {
						this.step -= 1
					} else if (prevLengthSpace === currLengthSpace) {
						const selectedValue = this.prevValue.slice(this.selection.s, this.selection.e)
						const isSpace = Array.isArray(selectedValue.match(/ /g))
						if (!isSpace) {
							this.step -= offsetLengthSpace
						}
						
						if (this.selection.s === 0 && formatLeftSide !== 0) {
							this.step = 0
						}
					} else {
						if (/^\d{1} /.test(target.value) || /^\d{2} /.test(target.value)) {
							this.step += 1
						}
					}
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
						const selectedValue = this.prevValue.slice(this.selection.s, this.selection.e)
						const isSpace = Array.isArray(selectedValue.match(/ /g))

						if (prevLengthSpace > currLengthSpace) {
							this.step -= 1
						} else if (prevLengthSpace < currLengthSpace) {
							if (/^\d{1} /.test(target.value) || /^\d{2} /.test(target.value)) {
								this.step += 1
							}
						} else {
							if (!isSpace) {
								this.step -= offsetLengthSpace
							}
						}
					} else {
						this.step = !this.isInteger && leftSideIndex === 1 && this.step <= 2
							? 1
							: this.step + offsetLengthSpace
					}
				} else {
					if (this.isSelectableValue) {
						if (!this.isDelimiter) {
							target.setSelectionRange(this.selection.s, this.selection.e)
							return
						} else {
							this.step = leftSideIndex + 1
						}
					} else {
						if (this.isDelimiter) {
							this.step = leftSideIndex + 1
						} else {
							this.step -= 1
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
				console.log('watch')
				const { decimal, value } = props,
						iDecimal = +decimal,
						iValue = this.initialValue(value, iDecimal),
						valueToNumber = Number(iValue.replace(/,/, '.'))

				this.iValue = this.separatorValue(iValue)
				this.isInteger = Number.isInteger(valueToNumber) && !iDecimal
				this.iDecimal = iDecimal
			}
		}
	},
}
</script>

<style lang="scss">

</style>