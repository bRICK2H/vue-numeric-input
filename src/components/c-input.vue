<template>
	<input type="text"
		:value="initValue"
		@input="input($event)"
		@keydown="keydown($event)"
	>
</template>

<script>
	export default {
		name: 'cInput',
		props: {
			value: {
				type: [Number, String],
				default: 0
			}
		},
		data: () => ({
			initValue: null,
			prevValue: null,
			valueSettings: {}
		}),
		methods: {
			input(e) {
				const { target } = e
				target.value = this.parseValue(target.value)
			},
			setTypeOfNumber(value) {
				const format = String(value).replace(/\./, ',')
				const isInteger = !(/,/.test(format))

				return {
					isInteger,
				}
			},
			parseValue(val) {
				/**
				 * 1. Нужно сразу же определить целое это число или дробное
				 * 2. В не зависимости того пришло число или строка, точка или запятая нужно преобразовать все к строке и запятой
				 * 3. 
				 */
				
				console.log(this.typeOfNumber)
				const format = String(val).replace(/[^\d\.,]/g, '').replace(/\./, ',')
				
				console.log({format}, format.match(/[,\.]/g), this.prevValue)

				// if (isInteger) {
				// 	console.log('целове число')
				// 	return format.replace(/,/, '')
				// } else {
				// 	if (format.match(/[,\.]/g) === null) {
				// 		console.log('нет запятых')
				// 	} else if(format.match(/[,\.]/g).length > 1) {
						
				// 		console.log('пришла точка или запятая', format.match(/[,\.]/), this.prevValue)
				// 		return this.prevValue
				// 	} else {
				// 		console.log('hs')
				// 		// return format.replace(/,/, '')
				// 		return format
				// 	}
				// 	console.log('дробное число')
				// }
				
				// console.log('val:', val)
				return format
			},
			keydown(e) {
				const { target } = e
				this.prevValue = target.value
			}

		},
		created() {
			this.typeOfNumber = this.setTypeOfNumber(this.value)
			this.initValue = this.parseValue(this.value)
		}
	}
</script>

<style lang="scss">
	input {
		padding: 10px;
	}
</style>