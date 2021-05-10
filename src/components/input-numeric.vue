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
			default: 0
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
		isGroupSelected: false,
		isInteger: false,
		isDelimiter: false,
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
					numericValue = Number(convertedValue.replace(/,/, '.'))

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
			console.log('enter converValue', value)
			const unSeparateValue = value.replace(/ /g, '')

			if (this.isInteger) {
				const clearValue = unSeparateValue.replace(/[^\d]/g, '')
				return Number(clearValue)
			} else {
				const isExistComma = /,/.test(value),
						isNumeric = Number(unSeparateValue.replace(/,/, '.'))

				if (this.isDelimiter && !this.isGroupSelected) {
					console.log('here?', value, this.step - 1)
					const arrValue = value.split('')
					arrValue.splice(this.step - 1, 1)

					return arrValue.join('').replace(/ /g, '')
				}

				if (!isExistComma) {
					console.log('!isExistComma', value)
					if (!this.isGroupSelected) {
						return this.prevValue.replace(/ /g, '')
					} else {
						if (isNaN(isNumeric)) {
							return this.prevValue.replace(/ /g, '')
						} 	else {
							const spaceCount = value.match(/ /g) ? value.match(/ /g).length : 0,
									clearValue = String(Number(value.replace(/[^\d]/g, ''))),
									arrValue = clearValue.split('')

							arrValue.splice(this.isDelimiter ? this.step - 1 : this.step - spaceCount, 0, '.')
							const fixedValue = Number(arrValue.join('').replace(/ /g, '')).toFixed(this.iDecimal)

							return fixedValue.replace(/\./, ',')
						}
					}
				}

				if(isNaN(isNumeric) && this.isGroupSelected && !this.isDelimiter) {
					return this.prevValue.replace(/ /g, '')
				}

				const clearValue = unSeparateValue.replace(/[^\d,]/g, '')
				const fixedValue = Number(clearValue.replace(/,/, '.')).toFixed(this.iDecimal)

				return fixedValue.replace(/\./, ',')
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
			const leftSide = target.value.length - this.iDecimal - 1,
					isOneNumeric = leftSide === 1 && this.step === 2 || !this.step,
					currOffset = target.value.length - target.value.replace(/ /g, '').length,
					resOffset = this.prevOffset === currOffset ? 0 : 1

			this.prevOffset = currOffset

			if(this.isDeletes) {
				if (this.isGroupSelected) {
					console.log('cursor del isGroup')
					
				} else {
					if(this.isBackspace) {
						this.step = isOneNumeric ? 1 : this.step - resOffset
					} else {
						const empty = ' '
						const prevLeftSide = this.prevValue.length - this.iDecimal - 1
	
						if (this.step && prevLeftSide > this.step) {
							this.step = this.prevValue[this.step] === empty ? this.step + 1 : this.step - resOffset
						} else if (/^0/.test(target.value) && prevLeftSide > this.step) {
							this.step = 1
						} else if (this.step === prevLeftSide) {
							this.step += 1
						}
					}
				}
			} else if (this.isNumeric) {
				if (this.isGroupSelected) {
					const getLastIndexOfThree = value => {
						return value.indexOf(' ') !== -1
							? value.indexOf(' ')
							: value.slice(0, 3).indexOf(',') !== -1
								? value.indexOf(',')
								: 3
					}
					console.log(target.value, this.prevValue)
					const pieceOfValue = target.value.slice(0, getLastIndexOfThree(target.value))
					const pieceOfPrevValue = this.prevValue.slice(0, getLastIndexOfThree(this.prevValue))
					const selectionValue = this.prevValue.slice(this.selection.s, this.selection.e)
					const spaceCount = selectionValue.match(/ /g)


					if (spaceCount) {
						console.log('has',
							spaceCount,
							{ prevSpace:  this.prevValue.match(/ /g) ? this.prevValue.match(/ /g).length : 0 },
							{ currSpace: target.value.match(/ /g) ? target.value.match(/ /g).length : 0 },
							{selection: this.selection.s },
							{step: this.step }
						)
						const pv = this.prevValue.match(/ /g) ? this.prevValue.match(/ /g).length : 0
						const v = target.value.match(/ /g) ? target.value.match(/ /g).length : 0
						const test = this.prevValue.slice(this.selection.s - 1, this.selection.e)[0]
						console.log(`'${test}'`)

						if(test === ' ' && (/^\d{2} /.test(target.value) || /^\d{3} /.test(target.value))) {
							console.log('inner')
							this.step -= 1
						} else if (/^\d{1} /.test(target.value) || pieceOfValue.length === pieceOfPrevValue.length) {
							console.log('1s') // в некоторых случаях дает лишний +1
							this.step += 1
						}
						

							// if (pv === v) {
							// 	console.log('==')
							// 	this.step += 1
							// }


						// this.step = this.selection.s
						// this.step = pieceOfValue.length < pieceOfPrevValue.length
						// 	? spaceCount.length === 1 ? this.step + 1 : this.step
						// 	: spaceCount.length === 1 ? this.step : this.step + 1
					} else {
						console.log('nt', spaceCount)
						this.step = pieceOfValue.length > pieceOfPrevValue.length
							? this.step - 1
							: this.step
					}
					// if(pieceOfValue.length < pieceOfPrevValue.length) {
					// 	console.log('???')
					// 	this.step += 1
					// } else if (!selectionValue.match(/ /g) && pieceOfValue.length > pieceOfPrevValue.length) {
					// 	console.log('!!!')
					// 	this.step -= 1
					// }
					console.log({pieceOfValue}, {pieceOfPrevValue}, {selectionValue})

				} else {
					this.step = isOneNumeric ? 1 : this.step + resOffset
				}
			} else if (this.isDelimiter) {
				this.step = leftSide + 1
			} else {
				this.step -= 1
			}

			target.setSelectionRange(this.step, this.step)
		},
		keyAction(e) {
			const { target } = e

			this.prevValue = target.value
			this.selection.s = target.selectionStart
			this.selection.e = target.selectionEnd
			this.isGroupSelected = this.selection.s !== this.selection.e,
			this.isDelimiter = ['Comma', 'Period'].includes(e.code)
			this.isDelete = e.code === 'Delete'
			this.isBackspace = e.code === 'Backspace'
			this.isDeletes = this.isDelete || this.isBackspace
			this.isNumeric = [0,1,2,3,4,5,6,7,8,9].map(curr => `Digit${curr}`).includes(e.code)
		},
	},
	watch: {
		$props: {
			deep: true,
			immediate: true,
			handler(props) {
				const { decimal, value } = props,
						iDecimal = +decimal,
						iValue = this.initialValue(value, iDecimal),
						valueToNumber = Number(iValue.replace(/,/, '.'))

				this.iValue = this.separatorValue(iValue)
				this.isInteger = Number.isInteger(valueToNumber) && !iDecimal
				this.iDecimal = iDecimal
			}
		}
	}
}
</script>

<style lang="scss">

</style>