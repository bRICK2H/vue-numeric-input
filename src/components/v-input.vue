<template>
	<input type="text"
		:value="initValue"
		@input="inputValue($event)"
		@focus="focus($event)"
		@focusout="unfocus"
		@keyup.esc="keyup($event)"
		@keydown="keydown($event)"
		@dblclick="isFocus = true"
		@click="clickCounter++"
	>
</template>

<script>
export default {
	name: 'vInput',
	props: {
		value: {
			type: [String, Number],
			default: 0,
		},
		maxNumAfterComma: {
			type: Number,
			default: 2
		}
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
		dedicatedSelection: []
	}),
	methods: {
		inputValue(e) {
			const { target } = e,
					isNumeric = !isNaN(Number(target.value))

			this.beforeSelection = target.selectionStart
			console.log(e, this.clickCounter)
			if (this.clickCounter >= 2) {
				this.isFocus = false
				this.clickCounter = 0
			}

			// Фокус
			if (this.isFocus) {
				console.log('focus: ')
				if (!isNumeric) {
					console.log('ne num')
					target.value = this.value
					target.select()
					return
				}

				console.log('if numeric')
				target.value = Number(target.value).toFixed(this.maxNumAfterComma)
				target.selectionStart = target.selectionEnd = 1
				this.isFocus = false
				this.$emit('input', Number(target.value))
				return
			}

			// Отсутствие дробной части
			if (!this.maxNumAfterComma) {
				console.log('isInt')
				if (!isNumeric) {
					target.value = target.value.replace(/[^\d]/g, '')
				}
				
				this.$emit('input', Number(target.value))
				return
			}
			

			// Дробная часть присутствует
			if (!isNumeric || this.isExhibitor) {
				console.log('isFloat', target.selectionStart)
				target.value = this.value.toFixed(this.maxNumAfterComma)
				target.setSelectionRange(...this.dedicatedSelection)
				// target.selectionStart = target.selectionEnd = this.beforeSelection

				if (this.isPoint || this.isComma) {
					console.log('.,')
					target.value = Number(target.value).toFixed(this.maxNumAfterComma)
					target.selectionStart = target.selectionEnd = target.value.indexOf('.') + 1
				}

				return
			}

			console.log('go') // Bug, запятая и точка ломает всю логику

			const [before, after] = target.value.split('.'),
					beforeCommaLength = before.length + 1

			/**
			 * If cursor contains comma
			 */
			if (after === undefined) {
				console.log('inner comma', target.value, this.value, this.dedicated)
				const curArrayValue = String(this.value).split(''),
						[beforeIndex] = this.dedicatedSelection
				curArrayValue.forEach((curr, i) => {
					if (this.dedicated.includes(i) && curr.indexOf('.') === -1) {
						curArrayValue.splice(i, 1, '')
					}
				})

				console.log(beforeIndex)
				if(e.data) curArrayValue.splice(beforeIndex, 1, e.data)
				if (isNaN(Number(curArrayValue.join('')))) {
					const zero = 0
					target.value = zero.toFixed(this.maxNumAfterComma)
				} else {
					target.value = Number(curArrayValue.join('')).toFixed(this.maxNumAfterComma)
				}

				target.selectionStart = target.selectionEnd = this.beforeSelection
				if (this.isDeleteForward) {
					target.selectionStart = target.selectionEnd = target.value.indexOf('.') + 1
				}
				this.$emit('input', Number(target.value))
				
				return

				// const [beforeOriginalValue] = String(this.value).split('.')
				// console.log('comma: ', target.code)
				// if (this.isBothDelete) {
				// 	console.log('inner comma')
				// 	const curArrayValue = String(this.value).split('')
				// 	curArrayValue.forEach((curr, i) => {
				// 		if (this.dedicated.includes(i) && curr.indexOf('.') === -1) {
				// 			curArrayValue.splice(i, 1, '')
				// 		}
				// 	})

				// 	if (isNaN(Number(curArrayValue.join('')))) {
				// 		const zero = 0
				// 		target.value = zero.toFixed(this.maxNumAfterComma)
				// 	} else {
				// 		target.value = Number(curArrayValue.join('')).toFixed(this.maxNumAfterComma)
				// 	}

				// 	target.selectionStart = target.selectionEnd = this.beforeSelection
				// 	this.$emit('input', Number(target.value))
					
				// 	return
				// } else {
				// 	console.log('addd', target.value, this.value, this.dedicated)
				// }
				// if (this.isDeleteForward) {
				// 	target.value = this.value.toFixed(this.maxNumAfterComma)
				// 	target.selectionStart = target.selectionEnd = target.value.indexOf('.') + 1
				// 	return
				// }

				// target.value = Number(beforeOriginalValue).toFixed(this.maxNumAfterComma)
				// target.selectionStart = target.selectionEnd = target.value.indexOf('.')
				// return
			}

			/**
			 * If cursor located in input
			 * left and right side
			 */
			if (target.selectionStart > before.length) {
				console.log('before', this.dedicated)

				// right side if deleted value
				if (!this.isBothDelete) {
					console.log('!del rigth')
					const afterArr = after.split('')
							// changedIndex = afterArr.findIndex((el, i) => i === target.selectionStart - beforeCommaLength)
	
					afterArr.splice(target.selectionStart - beforeCommaLength, 1)
					target.value = Number(`${before}.${afterArr.join('')}`).toFixed(this.maxNumAfterComma)
					this.$emit('input', Number(target.value))
					target.selectionStart = target.selectionEnd = this.beforeSelection
					// console.log(beforeCommaLength + changedIndex, this.beforeSelection)
	
					const [, afterUp] = target.value.split('.')
	
					if (afterUp.length > this.maxNumAfterComma) {
						afterArr.splice(-1, 1)
						target.value = `${before}.${afterArr.join('')}`
						this.$emit('input', Number(target.value))
						target.selectionStart = target.selectionEnd = target.value.length
					}
				} else {
					console.log('del right')
					// rigth side if added value
					const [, afterComma] = String(Number(target.value).toFixed(this.maxNumAfterComma)).split('.'),
							afterArr = afterComma.split(''),
							deletedIndex = afterArr.findIndex((el, i) => i === target.selectionStart - beforeCommaLength)
					console.log(beforeCommaLength + deletedIndex, this.beforeSelection)

					target.value = Number(target.value).toFixed(this.maxNumAfterComma)
					this.$emit('input', Number(target.value))
					target.selectionStart = target.selectionEnd = beforeCommaLength + deletedIndex
				}
				
			} else {
				console.log('left side toggle', target.value, {bS: this.beforeSelection, s: target.selectionStart})
				const isFirstZero = !Number(before[0])
				target.value = Number(target.value).toFixed(this.maxNumAfterComma)
				target.selectionStart = target.selectionEnd = this.beforeSelection

				if (isFirstZero && !this.isBothDelete) {
					target.selectionStart = target.selectionEnd = this.beforeSelection - 1
				}

				// left side if deleted lasted value
				if (before === '') {
					console.log('before del', after)
					target.value = `${Number(before)}.${after}`
					target.selectionStart = target.selectionEnd = target.value.indexOf('.')
				}

				this.$emit('input', Number(target.value))

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
			// const tmp = []
			for(let i = target.selectionStart; i < target.selectionEnd; i++) {
				this.dedicated.push(i)
			}
			// console.log(tmp)

			if (isArrowsExist || isComma || isPoint) {
				this.isFocus = false
			}
		},
		keyup(e) {
			const { target } = e
			target.value = this.prevValue
			this.$emit('input', this.prevValue)
		},
		focus(e) {
			const { target } = e
			console.log('focus')
			target.select()
			this.isFocus = true
		},
		unfocus(e) {
			const { target } = e
			this.prevValue = target.value
			this.isFocus = false
		}
	},
	created() {
		const outerValue = this.value
		
		if (!this.maxNumAfterComma) {
			this.initValue = Math.floor(outerValue)
		} else {
			const [, vAfterComma] = String(outerValue).split('.'),
					vAfterCommaLen = vAfterComma.length

			if (vAfterCommaLen >= this.maxNumAfterComma) {

				const outerValueArr = String(outerValue).split(''),
						outerCommaIndex = String(outerValue).indexOf('.') + 1

				this.initValue = outerValueArr.splice(0, this.maxNumAfterComma + outerCommaIndex).join('')
			} else {
				this.initValue = outerValue.toFixed(this.maxNumAfterComma)
			}
			
		}

		this.prevValue = this.initValue
	},
}
</script>

<style>

</style>