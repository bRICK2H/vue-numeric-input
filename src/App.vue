<template>
	<div class="container">
		<c-input
			:dataRef="data"
			field="value"
		/>

		outer: {{ data.value }}

		<v-input
			:dataRef="data"
			field="value"
			:maxNumAfterComma="2"
			:value="mValue"
			v-model="mValue"
		/>

		outer: {{ mValue }}
	</div>
</template>

<script>
import cInput from './components/c-input'
import vInput from './components/v-input'

export default {
	name: 'App',
	components: {
		cInput,
		vInput
	},
	data: () => ({
		data: { value: 2.420 },
		mValue: 3.25
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