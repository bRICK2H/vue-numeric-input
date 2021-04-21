<template>
	<input type="text"
		:value="toInput"
		@input="inputValue($event)"
	>
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
		
	}),
	computed: {
		toInput() {
			const value = String(this.value)
				.replace(/[^\d,\.]/g, '')
				.replace(/[,\.]/, '.')
			console.log(value)
			const [before, after] = value.split('.')
			const modifyBefore = before.split('')
			const newValue = []

			for (let i = modifyBefore.length; i > 0; i -= 3) {
				i - 3 >= 0
					? newValue.unshift(' ', ...modifyBefore.splice([i] - 3, 3))
					: newValue.unshift(...modifyBefore)
			}
			newValue.push('.')
			console.log(newValue.join('').trim().concat(after))
			return Number(newValue.join('').trim().concat(after)).toFixed(this.decimal)
		}
	},
	methods: {
		inputValue(e) {
			const { target: { value } } = e
			this.$emit('input', value)
		},
		parseValue() {
			
		}
	},
	created() {
		console.log(this.value, this.decimal)
	}
}
</script>

<style>

</style>