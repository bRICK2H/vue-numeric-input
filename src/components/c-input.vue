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
			selection: {},
			isInteger: false,
			isNumeric: false,
			isDeletes: false,
			isBackspace: false,
			isSeparator: false,
		}),
		methods: {
			input(e) {
				const { target } = e

				if (!(/,/.test(target.value)) && this.decimal) {
					console.log('comma')
					// const nArr = target.value.split(''),
					// 		pos = target.value.length - this.decimal - 1
					// nArr.splice(-this.decimal, 0, ',')
					// target.value = nArr.join('')

					target.value = this.prevValue
				}

				console.log('TV:', target.value)
				target.value = this.setSeparatorSpace(this.parseValue(target.value))
				// target.value = this.parseValue(target.value)

				// console.log(this.selection.s - 1, target.value.indexOf(','), target.value, target.selectionStart)
				// const l = target.value.slice(0, target.value.indexOf(','))
				// const r = target.value.slice(-(this.decimal))
				// console.log({r, l}, this.selection.s - 1, 'l:', l.length, 'tv:', target.value.length, 's: ', this.selection.s, target.selectionStart)
				// const o = {
				// 	left: l.split('').map((c, i) => i + 1),
				// 	right: r.split('').map((c, i) => i + 1 + r.length),
				// 	comma: [l.length + 1]
				// }

				// console.log({o}, this.selection.s)
				// let test = null
				// Object.entries(o).map(curr => {
				// 	const [side, pos] = curr
				// 	if (pos.includes(this.selection.s)) {
				// 		console.log({side})
				// 		test = side
				// 	}
				// })

				// let currSide = this.selection.s - 1 > target.value.length - this.decimal ? 'right' : 'left'
				const left = target.value.slice(0, target.value.indexOf(','))

				console.log({
					left,
					leftLength: left.length,
					targLength: target.value.length,
					leftSide: target.value.length - this.decimal,
					selection: this.selection.s,
					selectionPlus: this.selection.s + (left.length - left.replace(/ /g, '').length),
					// offset: (left.length - left.replace(/ /g, '').length),
				})

				/**
				 * отталкиваться от запятой чтоли хз +- работает
				 */
				// let currSide = this.selection.s > left.length + left.length - left.replace(/ /g, '').length ? 'right' : 'left'
				// console.log(target.value.slice(0, target.value.indexOf(',')).length, target.value.indexOf(','), this.selection.s) // i1,s2 / i2,s3 / i2,s4
				let currSide = target.value.length - 1 > this.selection.s ? 'left' : 'right'

				const isLeftExist = left.slice(0, 3) === left.replace(/ /g, '').slice(0, 3)
				console.log({currSide})

				if (this.isDeletes) {
					// console.log({test})
					if (this.isBackspace) {
						switch (currSide) {
							case 'left':
								const p1 =  /^\d{3}/.test(target.value) ? 1 : 0
								// const test = isLeftExist ? 1 : 0
								
								console.log('backw/left', p1)
								target.setSelectionRange(
									this.selection.s - 1 - p1,
									this.selection.e - 1 - p1,
									// this.selection.e - p1
									// this.selection.e - p1
								)
								break;
							case 'right':
								// const p =  /^\d{3}/.test(target.value) ? 1 : 0
								// const p2 =  /^\d{2} /.test(target.value) ? 1 : 0
								// const p3 =  /^\d{1} \d{1}/.test(target.value) ? 1 : 0
								// console.log('backw/right', p, p2, p3)
								console.log('backw/right')
								
								target.setSelectionRange(
									this.selection.s - 1,
									this.selection.e - 1
									// this.selection.s - p - p2 - p3,
									// this.selection.e - p - p2 - p3
								)
								console.log(this.selection.s, this.selection.e)
								break;
						}
					}
				} else {
					console.log('ELSE')
					// target.setSelectionRange(
					// 	this.selection.s - 1,
					// 	this.selection.e - 1
					// )
					switch (currSide) {
						case 'left':
							let offset = left.length - left.replace(/ /g, '').length
							const p = /^\d{1} /.test(target.value) ? 1 : 0
							// если пред один ноль
							const p2 = /^\d{1},/.test(target.value) ? 0 : 1
							console.log('add/left', p, p2)
							target.setSelectionRange(this.selection.s + p2 + p, this.selection.e + p2 + p)
							break;
						case 'right':
							// const rp = /^\d{1} /.test(target.value) ? 1 : 0
							target.setSelectionRange(this.selection.s + 1, this.selection.e + 1)
							console.log('add/right')
							break;
					}
				}

				

				// if (!this.isInteger) {
				// 	if (this.isDeletes) {
				// 		console.log('del', target.value)

				// 		// const pos = /^\d{1} /.test(target.value)
				// 		// 	? 1
				// 		// 	: 0
				// 		// 	console.log('isDeletes', pos, /^\d{3}/.test(target.value), currSide)
				// 		// if (this.isBackspace) {
				// 		// 	target.setSelectionRange(
				// 		// 		this.selection.s - 1,
				// 		// 		this.selection.e - 1
				// 		// 	)
				// 		// } else {
				// 		// 	console.log('backwa', /^0,\d+$/.test(target.value), target.value)
				// 		// 	const pos = /^0,\d+$/.test(target.value) ? 1 : 0
							
				// 		// 	target.setSelectionRange(this.selection.s + pos, this.selection.e + pos)
				// 		// }
				// 	} else {
				// 		if (this.isSeparator) {
				// 			// позиционирование для точки или запятой
				// 			const indexComma = target.value.indexOf(',') + 1
				// 			target.setSelectionRange(indexComma, indexComma)
				// 		} else if (this.isNumeric) {
				// 			// позиционирование только для добавления чисел
				// 			console.log('HERE: ', /^\d{1} /.test(target.value), target.value, /^\d{1},/.test(target.value))
				// 			if ( /^\d{1},/.test(target.value)) {
				// 				console.log('nul')
				// 				target.setSelectionRange(this.selection.s, this.selection.e)
				// 			} else {
				// 				const pos = /^\d{1} /.test(target.value) && currSide === 'left' ? 1 : 0
								
				// 				target.setSelectionRange(
				// 					this.selection.s + 1 + pos,
				// 					this.selection.e + 1 + pos
				// 				)
				// 			}
				// 		} else {
				// 			// вся белеберда кроме запятой, точки и числа
				// 			target.setSelectionRange(this.selection.s, this.selection.e)
				// 		}
				// 	}
				// } else {
				// 	console.log('here')
				// }
				
			},
			parseValue(val) {
				const sValue = String(val).replace(/ /g, '')
				console.log({sValue})

				if (this.isInteger) {
					return sValue.replace(/[^\d]/g, '')
				} else {
					let format =  sValue.replace(/[^\d\.,]/g, '').replace(/\./, ',')
					const countComma = format.match(/,/g)
					const unFormat = Number(format.replace(/,/, '.'))
					const fixedNumber = unFormat.toFixed(this.decimal)

					if (countComma && countComma.length > 1) {
						return this.prevValue.replace(/ /g, '')
					}

					if (this.decimal >= this.initDecimalLength) {
						return String(fixedNumber).replace(/\./, ',')
					} else {
						const sValue = String(fixedNumber).replace(/\./, ',')
						const indexComma = sValue.indexOf(',')
						return `${sValue.slice(0, indexComma)}${sValue.slice(indexComma, (indexComma + 1) + this.decimal)}`
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
				console.log('end: ', newArrValue.join('').trim())
				return newArrValue.join('').trim()
			},
			keydown(e) {
				const { target } = e
				this.prevValue = target.value
				this.isDelete = e.code === 'Delete',
				this.isBackspace = e.code === 'Backspace'
				this.isDeletes = this.isDelete || this.isBackspace,
				this.isSeparator = ['Comma', 'Period'].includes(e.code),
				this.isNumeric = [0,1,2,3,4,5,6,7,8,9].map(curr => `Digit${curr}`).includes(e.code),
				this.selection = { s: target.selectionStart, e: target.selectionEnd }

				// console.log(e.code)

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
		created() {
			const value = typeof this.value === 'number'
				? this.value
				: Number(this.value.replace(/,/, '.'))
			
			this.isInteger = this.decimal
				? Number.isInteger(value)
				: Number.isInteger(Math.floor(value))
			
			if (!this.isInteger) {
				const sValue = String(value)
				console.log({sV: sValue})
				this.initDecimalLength = sValue.slice(sValue.indexOf('.') + 1).length
			}
				
			this.initValue = this.decimal
				? this.setSeparatorSpace(this.parseValue(value))
				: this.setSeparatorSpace(this.parseValue(Math.floor(value)))
		}
	}
</script>

<style lang="scss">
	input {
		padding: 10px;
	}
</style>