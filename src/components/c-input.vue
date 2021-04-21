<template>
	<input type="text"
		:value="initValue"
		@input="input($event)"
		@focus="focus($event)"
		@blur="blur($event)"
		@keydown="keydown($event)"
		@dblclick="dblclick"
		@click="click($event)"
	>
</template>

<script>
	export default {
		name: 'cInput',
		props: {
			dataRef: {
				type: Object,
				default: () => ({})
			},
			field: {
				type: String,
				default: 0
			}
		},
		data: () => ({
			initValue: null,
			isFocus: false,
			isRemove: false,
			isNumeric: false,
			isComma: false,
			isDelete: false,
			countFocusOut: 0,
		}),
		methods: {
			input(e) {
				const { target } = e,
					innerValue = target.value.replace(/,/, '.'),
					isNumeric = !isNaN(innerValue)

				if (!isNumeric) {
					target.value = this.dataRef[this.field].toFixed(2).replace(/\./, ',')
					const [before] = target.value.split(',')
					target.selectionStart = target.selectionEnd = String(Number(before)).length
					if (this.isFocus) target.select()
					if (this.isComma) {
						target.selectionStart = target.selectionEnd = target.value.indexOf(',') + 1
						this.isComma = false
					}
					return
				} else if (this.isFocus) {
					target.value = Number(target.value).toFixed(2).replace(/\./, ',')
					this.dataRef[this.field] = Number(target.value.replace(/,/, '.'))
					target.selectionStart = target.selectionEnd = target.value.indexOf(',')
					this.isFocus = false
				}

				const [before, after] = target.value.split(',')
				this.countFocusOut = 0


				if (target.selectionStart <= target.value.indexOf(',')) {
					if (!target.selectionStart) {
						if (before === '') {
							target.value = `${Number(before)},${after}`
							target.selectionStart = target.selectionEnd = target.value.indexOf(',')
						} else {
							target.selectionStart = target.selectionEnd = 0
						}
					}

					if (before.length === 2 && before.indexOf(0) !== -1 && before[0] === '0') {
						const currNull = before.indexOf(0)
						const beforeArray = before.split('')
						beforeArray.splice(currNull, 1)
						target.value = `${beforeArray.join('')},${after}`
						target.selectionStart = target.selectionEnd = target.value.indexOf(',')
					}

					target.value = `${Number(before)},${after}`
					this.dataRef[this.field] = Number(target.value.replace(/,/, '.'))

				} else {
					if (after === undefined) {
						if(!target.selectionStart) {
							const zero = 0
							target.value = String(zero.toFixed(2)).replace(/\./, ',')
							this.dataRef[this.field] = Number(target.value.replace(/,/, '.'))
						} else {
							const tmp = target.value.split('')
							tmp.splice(target.selectionStart, 0, ',')
							target.value = Number(tmp.join('').replace(/,/, '.')).toFixed(2).replace(/\./, ',')
							this.dataRef[this.field] = Number(target.value.replace(/,/, '.'))
						}

						target.selectionStart = target.selectionEnd = this.isDelete
							? target.value.indexOf(',') + 1
							: target.value.indexOf(',')

						return
					}
					
					if (after.length < 2) {
						if (!((target.selectionStart - before.length) - 1)) {
							target.value = String(Number(target.value.replace(/,/, '.')).toFixed(2)).replace(/\./, ',')
							target.selectionStart = target.selectionEnd = target.value.indexOf(',') + 1
						} else {
							target.value = String(Number(target.value.replace(/,/, '.')).toFixed(2)).replace(/\./, ',')
							target.selectionStart = target.selectionEnd = target.value.indexOf(',') + 2
						}

					} else if (after.length > 2) {
						const afterToArray = after.split('')

						if (target.value.length - 2 === target.selectionStart) {
							afterToArray.splice(-2, 1)
							target.value = `${Number(before)},${afterToArray.join('')}`
							target.selectionStart = target.selectionEnd = target.value.length - 1
						} else if (target.value.length - 1 === target.selectionStart) {
							afterToArray.splice(-1, 1)
							target.value = `${Number(before)},${afterToArray.join('')}`
						} else if (target.value.length === target.selectionStart) {
							target.value = `${Number(before)},${after.slice(0, 2)}`
						}
					}

					if (after === '') {
						target.value = String(Number(before).toFixed(2)).replace(/\./, ',')
					}

					this.dataRef[this.field] = Number(target.value.replace(/,/, '.'))
				}
			},

			keydown(e) {
				const code = e.keyCode ? e.keyCode : e.which,
						isArrow = [37, 38, 39, 40].includes(code),
						isComma = 188 === code,
						isDelete = 46 === code

				this.isDelete = isDelete

				if (isArrow || isComma) {
					this.isFocus = false
					this.isComma = true
				}
			},
			focus(e) {
				const { target } = e,
						[before, after] = target.value.split(',')

				target.value = `${before.replace(/ /g, '')},${after}`
				this.isFocus = true
				target.select()
			},
			blur(e) {
				const { target } = e,
						[before, after] = target.value.split(',')

				target.value = `${this.setSeparatorSpace(before)},${after}`
				this.countFocusOut = 0
			},
			dblclick() {
				this.isFocus = true
			},
			click() {
				this.countFocusOut++
				if (this.countFocusOut > 1) {
					this.isFocus = false
				}
			},
			setSeparatorSpace(str) {
				let str2 = str.split('')
				const tmp = []
				for (let i = str2.length; i > 0; i -= 3) {
					i - 3 >= 0
						? tmp.unshift(' ', ...str2.splice([i] - 3, 3))
						: tmp.unshift(...str2)
				}

				return tmp.join('').trim()
			}
		},
		created() {
			this.initValue = this.dataRef[this.field].toFixed(2).replace(/\./, ',')
		}
	}
</script>

<style lang="scss">
	input {
		padding: 10px;
	}
</style>