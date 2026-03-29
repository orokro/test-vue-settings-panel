<!--
        Vector3Edit.vue
        ---------------

        A custom setting for editing 3D vector values (X, Y, Z).
-->
<script setup>
const props = defineProps({
	value: {
		type: Object,
		default: () => ({ x: 0, y: 0, z: 0 })
	},
	opts: {
		type: Object,
		default: () => ({})
	},
	lint: {
		type: Function,
		default: null
	},
	validate: {
		type: Function,
		default: null
	}
})

const emit = defineEmits(['change'])

const updateValue = (key, val) => {
	let newValue = val === '' ? 0 : Number(val)
	const updated = { ...props.value, [key]: newValue }

	if (props.lint) {
		props.lint(updated)
	}
	emit('change', updated)
}

const onBlur = () => {
	if (props.validate) {
		props.validate(props.value)
	}
}
</script>

<template>
	<div class="vector3-edit">
		<div class="vector-field">
			<span class="label">X:</span>
			<input type="number" :value="value.x" @input="updateValue('x', $event.target.value)" @blur="onBlur"
				class="vector-input" :step="opts.step || 1" />
		</div>
		<div class="vector-field">
			<span class="label">Y:</span>
			<input type="number" :value="value.y" @input="updateValue('y', $event.target.value)" @blur="onBlur"
				class="vector-input" :step="opts.step || 1" />
		</div>
		<div class="vector-field">
			<span class="label">Z:</span>
			<input type="number" :value="value.z" @input="updateValue('z', $event.target.value)" @blur="onBlur"
				class="vector-input" :step="opts.step || 1" />
		</div>
	</div>
</template>

<style lang="scss" scoped>
.vector3-edit {
	display: flex;
	gap: 12px;
	width: 100%;

	.vector-field {
		display: flex;
		align-items: center;
		gap: 4px;
		flex: 1;

		.label {
			font-size: 11px;
			font-weight: bold;
			color: #888;
			opacity: 0.7;
		}

		.vector-input {
			width: 100%;
			padding: 6px 8px;
			border-radius: 4px;
			border: 1px solid var(--input-border-color, #ccc);
			background-color: var(--input-bg-color, #fff);
			color: var(--input-text-color, #333);
			font-size: 13px;
			outline: none;
			transition: border-color 0.2s;

			&:focus {
				border-color: var(--input-focus-border-color, #4caf50);
			}
		}
	}
}
</style>
