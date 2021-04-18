<template>
	<input type="text"
		:value="initValue"
		@input="input($event)"
		@focus="focus($event)"
		@keydown="keydown($event)"
		@dblclick="dblclick($event)"
	>
		<!-- @click="click($event)" -->
		<!-- @blur="blur" -->
</template>

<script>
export default {
	name: 'cInput',
	props: {
		dataRef: {
			type: Object,
			default: () => ({})
		},
		field: {
			type: String,
			default: 0
		}
	},
	data: () => ({
		initValue: null,
		isFocus: false,
		isRemove: false,
		isNumeric: false,
		// countFocusOut: 0,
	}),
	methods: {
			/**
			 * ! ПОЕБЕНЬ
			 *  - 1. фокус, далее клик (твориться хуй пойми чего) т.к. фокус еще включен
			 */
			input(e) {
				const { target } = e,
					innerValue = target.value.replace(/,/, '.'),
					isNumeric = !isNaN(innerValue)

				/**
				 * * Если не число
				 * 	- возвращаем в инпут прерыдущее значение в нужном формате
				 * * Если фокус и вводится не число
				 *  	- оставляем фокус
				 *  - завершаем операцию
				 * * Или если ввелось число при фокусе
				 * 	- формируем нужное число для инпута
				 * 	- разделяем число на числа до и после запятой
				 * 	- устанавливаем нужную позицию для курсора
				 * 	- возвращаем число внешнему объектку
				 */

				if (!isNumeric) {
					target.value = this.dataRef[this.field].toFixed(2).replace(/\./, ',')
					const [before] = target.value.split(',')
					target.selectionStart = target.selectionEnd = String(Number(before)).length
					if (this.isFocus) target.select()
					return
				} else if (this.isFocus)  {
					target.value = Number(target.value).toFixed(2).replace(/\./, ',')
					const [before] = target.value.split(',')
					console.log('число и фокус', {before}, this.dataRef[this.field])
					this.dataRef[this.field] = Number(target.value.replace(/,/, '.'))
					console.log({before})
					target.selectionStart = target.selectionEnd = target.value.indexOf(',')
				}

				// * Выключаем фокус
				this.isFocus = false
				const [before, after] = target.value.split(',')

				if (target.selectionStart <= target.value.indexOf(',')) {
					console.log('USED LEFT SIDE', before, target.selectionStart, target.selectionEnd)

					/**
					 * * Если ставим чило перед первым числом и это число 0
					 * 	- заменяем на число
					 */
					if (target.selectionStart === 1) {
						console.log('before 0')
						// target.selectionStart = target.selectionEnd = 1
						console.log(!Number(before[before.length - 1]), before[0])
						if (!Number(before[before.length - 1]) && before[0]) {
							console.log('if 0')
							target.value = `${before[0]},${after}`
							console.log('target.value', target.value)
							// target.selectionStart = target.selectionEnd = target.value.indexOf(',')
							return
						}
					} else {
						console.log('after 0')
						target.value = `${Number(before)},${after}`
						// target.selectionStart = target.selectionEnd = target.value.indexOf(',')
						this.dataRef[this.field] = Number(target.value.replace(/,/, '.'))
					}
					

				} else {
					console.log('USED RIGHT SIDE', after)

					if (after.length < 2) {
						console.log('< 2 толком пока не делал даже')
						target.value += 0
						return
					} else if (after.length > 2) {
						const afterToArray = after.split('')

						if (target.value.length - 2 === target.selectionStart) {
							afterToArray.splice(-2, 1) // [1,2]
							target.value = `${Number(before)},${afterToArray.join('')}`
							target.selectionStart = target.selectionEnd = target.value.length - 1
						} else if(target.value.length - 1 === target.selectionStart) {
							afterToArray.splice(-1, 1)
							target.value = `${Number(before)},${afterToArray.join('')}`
						} else if (target.value.length === target.selectionStart) {
							target.value = `${Number(before)},${after.slice(0,2)}`
						}

						this.dataRef[this.field] = Number(target.value.replace(/,/, '.'))
						return
					}

				}



				// target.value = `${Number(before)},${after}`
				// this.dataRef[this.field] = Number(target.value.replace(/,/, '.'))

				// target.selectionStart = target.selectionEnd = String(Number(before)).length
			},
		
		keydown(e) {
			const { target } = e
				// charCode = e.which ? e.which : e.keyCode
			// this.isRemove = e.code === 'Backspace' || charCode === 8,
			// this.isNumeric = !(charCode != 43 && charCode > 31 && (charCode < 48 || charCode > 57))
			
		},
		focus(e) {
			const { target } = e
			this.isFocus = true
			target.select()
		},
		dblclick(e) {
			const { target } = e

			console.log('dbl', e.target.value, target.selectionStart)
		}
		// click(e) {
		// 	this.countFocusOut++
		// 	console.log('click', this.countFocusOut)
		// 	if (this.countFocusOut >= 1) {
		// 		this.isFocus = false
		// 	}
		// 	const { target } = e
		// },
		// blur() {
		// 	console.log('blur')
		// 	this.countFocusOut = 0
		// }
	},
	created() {
		this.initValue = this.dataRef[this.field].toFixed(2).replace(/\./, ',')
	}
}
</script>

<style lang="scss">
	input {
		padding: 10px;
	}
</style>