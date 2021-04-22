<template>
	<input type="text"
		:value="initValue"
		@input="inputValue($event)"
		@keydown="trigger($event)"
	>
		<!-- @keyup="trigger($event)" -->
</template>

<script>
export default {
	name: 'nInput',
	props: {
		decimal: {
			type: [String, Number],
			default: 0
		},
		value: {
			type: [String, Number],
			default: 0
		}
	},
	data: () => ({
		initValue: null,
		selection: [],
		isDeletes: false,
		isBackspace: false
	}),
	methods: {
		inputValue(e) {
			const { target } = e

			console.log('valBefore', {
				tv: target.value,
				index: this.selection[0] - 1,
				el: target.value[this.selection[0] - 1],
				sel: target.selectionStart
			})

			console.log(target.selectionStart <= target.value.length - this.decimal - 1 ? 'left' : 'rigth')
			if (!/,/.test(target.value) && this.decimal) {
				console.log('comma')
				const nArr = target.value.split(''),
						pos = target.value.length - this.decimal - 1
				nArr.splice(-this.decimal, 0, ',')
				target.value = nArr.join('')
			}
			
			target.value = this.toInput(target.value)
			
			console.log('valAfter', {
				tv: target.value,
				index: this.selection[0] - 1,
				el: target.value[this.selection[0] - 1]
			})


			if (this.isDeletes) {
				const pos = /^\d{3}$/g.test(target.value.slice(0, 3)) ? 1 : 0
				console.log('isDeletes')
				if (this.isBackspace) {
					target.setSelectionRange(
						this.selection[0] - 1 - pos,
						this.selection[1] - 1 - pos
					)
				} else {
					target.setSelectionRange(...this.selection)
				}
			} else {
				const pos = /^\d $/.test(target.value.slice(0, 2)) ? 1 : 0
				console.log('add: ', pos)
				target.setSelectionRange(
					this.selection[0] + 1 + pos,
					this.selection[1] + 1 + pos
				)
			}

			// this.$emit('input', this.toEmit(target.value))
		},
		toEmit(val) {
			return Number(val.replace(/ /g, '').replace(/,/, '.'))
		},
		toInput(val) {
			// console.log({val})
			const value = String(val)
				.replace(/[^\d,\.]/g, '')
				.replace(/[,\.]/, ',')

			let parseValue = null
			
			if (!this.decimal) {
				parseValue = value.indexOf(',') !== -1
					? value.slice(0, value.indexOf(','))
					: value
				console.log('!dec', parseValue)
			} else {
				const afterComma = value.slice(value.indexOf(',') + 1);
				console.log('dec', value, afterComma, String(Number(value.replace(/,/, '.')).toFixed(this.decimal)).replace(/\./, ',') )
				parseValue = Number(afterComma) > this.decimal
					? String(Number(value.replace(/,/, '.')).toFixed(this.decimal)).replace(/\./, ',')
					: value.slice(0, -(this.decimal + 1))
			}

			// console.log({parseValue})
			// return
		
			const newArrValue = []
			const arrValue = parseValue.split('')
			console.log({arrValue})
			let wholePart = arrValue.indexOf(',') !== -1
				? arrValue.splice(0, arrValue.indexOf(','))
				: arrValue.splice(0)
			// на выходе вроде все ок, но при удалении шляпа

			// console.log({wholePart, arrValue})
			// return

			for (let i = wholePart.length; i > 0; i -= 3) {
				i - 3 >= 0
					? newArrValue.unshift(' ', ...wholePart.splice([i] - 3, 3))
					: newArrValue.unshift(...wholePart)
			}
			newArrValue.push(...arrValue)
			console.log('preres: ', newArrValue.join('').trim())
			return newArrValue.join('').trim()
		},
		trigger(e) {
			const { target } = e
			this.selection = [target.selectionStart, target.selectionEnd]
			this.isDeletes = ['Backspace', 'Delete'].includes(e.code)
			this.isBackspace = e.code === 'Backspace'
			// target.setSelectionRange(...this.selection)
			console.log(e.type, target.selectionStart, target.selectionEnd, e.code)
		}
	},
	created() {
		this.initValue = this.toInput(this.value)
	}
}
</script>

<style>

</style>