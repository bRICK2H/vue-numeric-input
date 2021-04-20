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
		maxAfterComma: {
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
					isNumeric = !isNaN(Number(target.value)),
					isPoint = data === '.',
					isE = data === 'e',
					isDelete = ['deleteContentBackward', 'deleteContentForward'].includes(inputType)

			console.log({isNumeric, e, isDelete})


			// Отсутствие дробной части
			if (!this.maxAfterComma) {
				if (!isNumeric) {
					target.value = target.value.replace(/[^\d]/g, '')
				}
				
				this.$emit('input', Number(target.value))
				return
			}
			

			// Дробная часть присутствует
			if (!isNumeric || isE || !this.maxAfterComma) {
				target.value = this.dataRef[this.field]
				target.selectionStart = target.selectionEnd = target.value.indexOf('.')
				return
			}
			
			if (isPoint) {
				target.selectionStart = target.selectionEnd = target.value.indexOf('.') + 1
			}

			const [before, after] = target.value.split('.'),
					beforeCommaLength = before.length + 1
			console.log({before, after})

			// if comma
			if (after === undefined) {
				const [beforeOrigin] = String(this.dataRef[this.field]).split('.')
				target.value = Number(beforeOrigin).toFixed(this.maxAfterComma)
				target.selectionStart = target.selectionEnd = target.value.indexOf('.')
			}

			// if left
			if (target.selectionStart > before.length) {
				console.log('before')
				if (!isDelete) {
					console.log('!del')
					const afterArr = after.split(''),
							changedIndex = afterArr.findIndex((el, i) => i === target.selectionStart - beforeCommaLength)
	
					afterArr.splice(target.selectionStart - beforeCommaLength, 1)
					target.value = `${before}.${afterArr.join('')}`
					this.dataRef[this.field] = Number(target.value)
					target.selectionStart = target.selectionEnd = beforeCommaLength + changedIndex 
	
					const [, afterUp] = target.value.split('.')
	
					if (afterUp.length > this.maxAfterComma) {
						afterArr.splice(-1, 1)
						target.value = `${before}.${afterArr.join('')}`
						this.dataRef[this.field] = Number(target.value)
						target.selectionStart = target.selectionEnd = target.value.length
					}
				} else { // if right
					const [, afterComma] = String(Number(target.value).toFixed(this.maxAfterComma)).split('.'),
							afterArr = afterComma.split(''),
							deletedIndex = afterArr.findIndex((el, i) => i === target.selectionStart - beforeCommaLength)

					target.value = Number(target.value).toFixed(this.maxAfterComma)
					target.selectionStart = target.selectionEnd = beforeCommaLength + deletedIndex
				}
				
			} else {
				console.log('after')

				// if del
				if (before === '') {
					console.log('before del')
				}

			}
			
		},
	},
	watch: {
		eventPos(val) {
			console.log('watch:', val)
		}
	},
	created() {
		const outerValue = this.dataRef[this.field]
		
		if (!this.maxAfterComma) {
			this.initValue = Math.floor(outerValue)
		} else {
			const [, vAfterComma] = String(outerValue).split('.'),
					vAfterCommaLen = vAfterComma.length

			if (vAfterCommaLen >= this.maxAfterComma) {

				const outerValueArr = String(outerValue).split(''),
						outerCommaIndex = String(outerValue).indexOf('.') + 1

				this.initValue = outerValueArr.splice(0, this.maxAfterComma + outerCommaIndex).join('')
			} else {
				this.initValue = this.dataRef[this.field].toFixed(this.maxAfterComma)
			}
			
		}
	},
}
</script>

<style>

</style>