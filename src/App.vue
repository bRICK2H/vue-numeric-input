<template>
	<div class="container">
		<c-input
			:dataRef="data"
			field="value"
		/>

		outer: {{ data.value }}
	</div>
</template>

<script>
import cInput from './components/c-input'

export default {
	name: 'App',
	components: {
		cInput
	},
	data: () => ({
		data: { value: 1.4 }
	})
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