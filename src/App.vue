<template>
	<div class="container">
		<input-numeric
			:separator="true"
			:decimal="decimal"
			v-model="cValue"
		/>
	
		outer: {{ cValue }}
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
		data: { value: 2.420 },
		cValue: 4,
		decimal: 2,
		mValue: 154655,
		kValue: 154655.315,
		nValue: 1.44,
	}),
	watch: {
		'data.value'(val) {
			console.log('outer: ', val)
		},
		mValue(val) {
			console.log('val', val)
		}
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