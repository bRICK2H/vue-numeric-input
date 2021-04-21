<template>
	<div class="container">
		<c-input
			:dataRef="data"
			field="value"
		/>

		outer: {{ data.value }}

		<v-input
			:maxNumAfterComma="2"
			:value="mValue"
			v-model="mValue"
		/>

		outer: {{ mValue }}

		<!-- <k-input
			:value="kValue"
			:decimal="4"
		/> -->

		<n-input
			v-model="nValue"
			:decimal="2"
			:value="nValue"
		/>

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
		mValue: 3.25,
		kValue: 154655.315,
		nValue: 2000000.5945,
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