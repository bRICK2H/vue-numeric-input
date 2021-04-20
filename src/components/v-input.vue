<template>
	<input type="text"
		:value="initValue"
		@input="inputValue($event)"
	>
</template>

<script>
export default {
	name: 'vInput',
	props: {
		dataRef: {
			type: Object,
			default: () => ({})
		},
		field: {
			type: String,
			default: 0
		},
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
	}),
	methods: {
		inputValue(e) {
			const { target, data, inputType } = e,
					isComma = data === ',',
					isPoint = data === '.',
					isExhibitor = data === 'e',
					isNumeric = !isNaN(Number(target.value)),
					isDelete = ['deleteContentBackward', 'deleteContentForward'].includes(inputType)

			console.log({isNumeric, e, isDelete})


			// Отсутствие дробной части
			if (!this.maxNumAfterComma) {
				if (!isNumeric) {
					target.value = target.value.replace(/[^\d]/g, '')
				}
				
				this.$emit('input', Number(target.value))
				return
			}
			

			// Дробная часть присутствует
			if (!isNumeric || isExhibitor) {
				target.value = this.value
				target.selectionStart = target.selectionEnd = target.value.indexOf('.')

				if (isPoint || isComma) {
					target.selectionStart = target.selectionEnd = target.value.indexOf('.') + 1
				}

				return
			}

			console.log('go') // Bug, запятая и точка ломает всю логику

			const [before, after] = target.value.split('.'),
					beforeCommaLength = before.length + 1
			console.log({before, after})

			/**
			 * If cursor contains comma
			 */
			if (after === undefined) {
				console.log('comma: ')
				const [beforeOriginalValue] = String(this.value).split('.')
				target.value = Number(beforeOriginalValue).toFixed(this.maxNumAfterComma)
				target.selectionStart = target.selectionEnd = target.value.indexOf('.')
			}

			/**
			 * If cursor locatec in input
			 * left and right side
			 */
			if (target.selectionStart > before.length) {
				console.log('before')

				// right side if deleted value
				if (!isDelete) {
					console.log('!del')
					const afterArr = after.split(''),
							changedIndex = afterArr.findIndex((el, i) => i === target.selectionStart - beforeCommaLength)
	
					afterArr.splice(target.selectionStart - beforeCommaLength, 1)
					target.value = `${before}.${afterArr.join('')}`
					this.$emit('input', Number(target.value))
					target.selectionStart = target.selectionEnd = beforeCommaLength + changedIndex 
	
					const [, afterUp] = target.value.split('.')
	
					if (afterUp.length > this.maxNumAfterComma) {
						afterArr.splice(-1, 1)
						target.value = `${before}.${afterArr.join('')}`
						this.$emit('input', Number(target.value))
						target.selectionStart = target.selectionEnd = target.value.length
					}
				} else {
					// rigth side if added value
					const [, afterComma] = String(Number(target.value).toFixed(this.maxNumAfterComma)).split('.'),
							afterArr = afterComma.split(''),
							deletedIndex = afterArr.findIndex((el, i) => i === target.selectionStart - beforeCommaLength)

					target.value = Number(target.value).toFixed(this.maxNumAfterComma)
					this.$emit('input', Number(target.value))
					target.selectionStart = target.selectionEnd = beforeCommaLength + deletedIndex
				}
				
			} else {

				// left side toggle value
				const [beforeArrayValue] = target.value.split('.'),
					isInt = !!Number(beforeArrayValue[beforeArrayValue.length - 1]),
					isZero = !Number(beforeArrayValue[beforeArrayValue.length - 2]),
					isNull = beforeArrayValue[beforeArrayValue.length - 3] === undefined

				// replace zero before int
				if (isInt && isZero && isNull && !isDelete) {
					console.log('zero before')
					target.value = Number(target.value).toFixed(this.maxNumAfterComma)
					target.selectionStart = target.selectionEnd = target.value.indexOf('.')
				}

				// left side if deleted lasted value
				if (before === '') {
					console.log('before del')
					const zero = 0
					target.value = `${zero.toFixed(this.maxNumAfterComma)}`
					target.selectionStart = target.selectionEnd = target.value.indexOf('.')
				}

				this.$emit('input', Number(target.value))

			}
			
		},
	},
	watch: {
		eventPos(val) {
			console.log('watch:', val)
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
	},
}
</script>

<style>

</style>