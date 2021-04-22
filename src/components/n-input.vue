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
			console.log('val: ', val)
			const value = typeof val === 'string'
				? Number(val.replace(/[^\d,]/g, '').replace(/,/, '.'))
				: val

			let tmp = this.decimal
				? String(value).split('')
				: String(value)

			if (this.decimal) {
				tmp.splice(String(value).indexOf('.') + 1 + this.decimal)
			}
			console.log({tmp})
			const checkedDecimal = this.decimal
				? tmp.join('')
				: Math.floor(tmp)
			const arrValue = checkedDecimal.split('')
			const wholePart = arrValue.indexOf(',') !== -1
				? arrValue.splice(0, arrValue.indexOf(','))
				: arrValue

			console.log({wholePart}, arrValue.indexOf(','), arrValue.indexOf(',') !== -1)
			const newArrValue = []
			console.log({newArrValue})

			for (let i = wholePart.length; i > 0; i -= 3) {
				i - 3 >= 0
					? newArrValue.unshift(' ', ...wholePart.splice([i] - 3, 3))
					: newArrValue.unshift(...wholePart)
			}
			newArrValue.push(...arrValue)
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