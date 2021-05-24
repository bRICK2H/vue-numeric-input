<template>
	<div class="container">
		<input-numeric
			currency="ru"
			:separator="true"
			v-model="value"
			:decimal="decimal"
		/>
	
		outer: {{ value }}
	</div>
</template>

<script>
import inputNumeric from './components/input-numeric'

export default {
	name: 'App',
	components: {
		inputNumeric
	},
	data: () => ({
		value: 0,
		decimal: 2,
		val: 0
	}),
	watch: {
		'data.value'(val) {
			console.log('outer: ', val)
		},
		mValue(val) {
			console.log('val', val)
		}
	},
	created() {
		setTimeout(() => this.value = 1000, 400)
	}
}
</script>

<style lang="scss">
	* {
		margin: 0;
		padding: 0;
	}
	.container {
		height: 100vh;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
</style>