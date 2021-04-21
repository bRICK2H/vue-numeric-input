<template>
	<input type="text"
		ref="vv"
		@focus="ifocus($event)"
		@input="ichange($event)"
		@keyup="keyup($event)"
		@mouseup="mouseup($event)"
		decimal="4"
		:value="nformat(value)"
	/>
</template>

<script>
export default {
	name: 'kInput',
	data: function() {
		return {
			start: this.parseValue(this.value),
			before: this.start,
			dec: parseInt(this.decimal),
			selection: []
		}
	},
	props: {
		value: {
			type: [String, Number],
			default: 0
		},
		decimal: {
			type: [String, Number],
			default: 0
		}
	},
	methods: {
		ichange(e) {
			const { target } = e,
					r = /^\\d{1,15},\\d{${this.dec},${this.dec}}$/,
					reg = new RegExp(r),
					val = target.value.replace(/[^\d,]/g, ''),
					res = val.match(reg)
					console.log('reg: ', val)

			if (res) {
				console.log('res', res)
				this.selection = this.getSelection(target)
				this.before = this.parseValue(target.value);
			} else {
				// Проверка на удаление запятой
				if (val.match(/\d/)) {
					console.log('coma deleted');

					console.log('change', this.selection[0]);
					if (this.selection[0] < target.value.length - this.dec - 1) {
						console.log('before');
					} else {
						console.log('after');
					}

				}
				// ----------------------------
			}

			target.value = this.nformat(this.before)
			console.log(target.value)
			target.setSelectionRange(this.selection[0], this.selection[1]);

			// const { target } = e,
			// 		val = target.value.replace(/[^\d,]/g, '')
			// this.selection = this.getSelection(target)

			// console.log(target.value, val, this.before, this.start)
			// // this.before = this.parseValue(val);
			// target.value = val;
			// target.setSelectionRange(this.selection[0], this.selection[1])
			
		},
		ifocus(e) {
			const { target } = e
			console.log('focus');
			this.before = this.start = this.parseValue(target.value)
			target.select()
		},
		keyup(e) {
			const { target } = e
			if (e.code == 'Escape') {
				console.log(this.start, this.nformat(this.start))
				target.value = this.nformat(this.start)
				target.select()
			}
			this.selection = this.getSelection(target);
			console.log('keyup', this.selection);
		},
		mouseup(e) {
			const { target } = e
			this.selection = this.getSelection(target);
			console.log('mouseup', this.selection)
		},
		parseValue(val) {
			const clear = String(val).replace(/[^\d,\.]/g, '').replace(/,/g, '.')
			return parseFloat(clear);
		},
		nformat(val) {
			return new Intl.NumberFormat('ru-RU', {
				minimumFractionDigits: this.dec,
				maximumFractionDigits: this.dec,
				useGrouping: true
			}).format(val);
		},
		getSelection(el) {
			if(!el) return
			return [el.selectionStart, el.selectionEnd];
		}
	},
}
</script>

<style>

</style>