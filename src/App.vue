<template>
	<div class="container">
		<input-numeric
			currency="ru"
			:separator="true"
			v-model="value"
			decimal="2"
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
		value: null,
		decimal: 2,
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
		setTimeout(() => this.value = 300, 400)
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