<template>
	<div class="container">
		<input-numeric
			:separator="true"
			:decimal="decimal"
			v-model="cValue"
		/>
		
		<!-- <c-input
			:decimal="decimal"
			v-model="cValue"
		/> -->
			<!-- :value="cValue" -->

		outer: {{ cValue }}

		<h3>v-i</h3>
		<v-input
			:decimal="0"
			:value="mValue"
			v-model="mValue"
			:width="'100%'"
		/>

		outer: {{ mValue }}

		<!-- <k-input
			:value="kValue"
			:decimal="4"
		/> -->

		<h3>n-i</h3>
		<!-- <n-input
			v-model="nValue"
			:decimal="2"
		/> -->

		input: {{ nValue }}
	</div>
</template>

<script>
import inputNumeric from './components/input-numeric'
import cInput from './components/c-input'
import vInput from './components/v-input'
import kInput from './components/k-input'
import nInput from './components/n-input'

export default {
	name: 'App',
	components: {
		cInput,
		inputNumeric,
		vInput,
		kInput,
		nInput,
	},
	data: () => ({
		data: { value: 2.420 },
		cValue: 123456789.23,
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