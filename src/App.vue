<template>
	<div class="container">
		<c-input
			:value="cValue"
			v-model="cValue"
		/>

		outer: {{ data.value }}

		<h3>v-i</h3>
		<v-input
			:decimal="1"
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
			:decimal="3"
		/> -->

		input: {{ nValue }}
	</div>
</template>

<script>
import cInput from './components/c-input'
import vInput from './components/v-input'
import kInput from './components/k-input'
import nInput from './components/n-input'

export default {
	name: 'App',
	components: {
		cInput,
		vInput,
		kInput,
		nInput
	},
	data: () => ({
		data: { value: 2.420 },
		cValue: 1.254,
		mValue: 154655.12,
		kValue: 154655.315,
		nValue: 12.44,
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