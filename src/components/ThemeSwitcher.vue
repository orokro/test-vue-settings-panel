<!--
        ThemeSwitcher.vue
        -----------------

        A utility to switch between different theme presets for the options panel.
-->
<script setup>
const props = defineProps({
	currentTheme: {
		type: Object,
		required: true
	},
	showSidebar: {
		type: Boolean,
		default: true
	}
})

const emit = defineEmits(['update:theme', 'update:showSidebar', 'update:spec'])

const themes = {
	light: {
		leftColumn: {
			bgColor: '#f5f5f5',
			categoryColor: '#333',
			selectedCategoryBgColor: '#e0e0e0',
			searchBgColor: '#fff',
		},
		mainColumn: {
			bgColor: '#fff',
			textColor: '#333',
			categoryHeaderColor: '#222',
			categoryHeaderTextColor: '#fff',
			attentionColor: '#4caf50',
		},
		range: {
			thumbColor: '#4caf50'
		}
	},
	dark: {
		leftColumn: {
			bgColor: '#1e1e1e',
			categoriesBoxBgColor: '#252526',
			categoriesBoxBorder: '1px solid #333',
			categoryColor: '#cccccc',
			categoryTextColor: '#cccccc',
			selectedCategoryBgColor: '#37373d',
			selectedCategoryTextColor: '#ffffff',
			searchBgColor: '#3c3c3c',
			searchXColor: '#999',
			searchTextColor: '#fff',
		},
		mainColumn: {
			bgColor: '#121212',
			textColor: '#e0e0e0',
			categoryHeaderColor: '#333',
			categoryHeaderTextColor: '#00abae',
			categoryBorder: '1px solid #333',
			categoryBgColor: '#1e1e1e',
			categoryTextColor: '#ccc',
			settingsRowBgColor: 'transparent',
			settingsRowNameColor: '#fff',
			settingsRowDescColor: '#888',
			settingsRowBorder: '1px solid #2d2d2d',
			subcategoryHeaderColor: '#252526',
			subcategoryHeaderTextColor: '#00abae',
			subcategoryBorder: 'none',
			subcategoryBgColor: '#1a1a1a',
			subCategoryTextColor: '#aaa',
			attentionColor: '#bb86fc',
		},
		toggle: {
			bgColor: '#444',
			thumbColor: '#ccc',
			activeBgColor: '#00abae'
		},
		input: {
			borderColor: '#444',
			bgColor: '#252526',
			textColor: '#fff',
			focusBorderColor: '#00abae'
		},
		range: {
			thumbColor: '#00abae'
		}
	},
	fun: {
		leftColumn: {
			bgColor: '#e0f7f7',
			categoriesBoxBgColor: '#f0ffff',
			categoriesBoxBorder: '2px dashed #00abae',
			categoryColor: '#007072',
			categoryTextColor: '#007072',
			selectedCategoryBgColor: '#00abae',
			selectedCategoryTextColor: '#ffffff',
			searchBgColor: '#ffffff',
			searchXColor: '#00abae',
			searchTextColor: '#004d40',
		},
		mainColumn: {
			bgColor: '#f9ffff',
			textColor: '#004d40',
			categoryHeaderColor: '#00abae',
			categoryHeaderTextColor: '#ffffff',
			categoryBorder: '2px solid #00abae',
			categoryBgColor: '#ffffff',
			categoryTextColor: '#004d40',
			settingsRowBgColor: '#f0fdfd',
			settingsRowNameColor: '#007072',
			settingsRowDescColor: '#00abae',
			settingsRowBorder: '1px solid #b2ebf2',
			subcategoryHeaderColor: '#e0f2f1',
			subcategoryHeaderTextColor: '#007072',
			subcategoryBorder: '1px dotted #00abae',
			subcategoryBgColor: '#ffffff',
			subCategoryTextColor: '#004d40',
			attentionColor: '#007072',
		},
		toggle: {
			bgColor: '#b2ebf2',
			thumbColor: '#ffffff',
			activeBgColor: '#00abae'
		},
		input: {
			borderColor: '#00abae',
			bgColor: '#ffffff',
			textColor: '#004d40',
			focusBorderColor: '#004d40'
		},
		range: {
			thumbColor: '#007072'
		}
	}
}

const selectTheme = (name) => {
	emit('update:theme', themes[name])
}

const toggleSidebar = () => {
	emit('update:showSidebar', !props.showSidebar)
}

const selectSpec = (name) => {
	emit('update:spec', name)
}
</script>

<template>
	<div class="header-toolbar">
		<div class="toolbar-section">
			<span class="label">Theme:</span>
			<div class="buttons">
				<button @click="selectTheme('light')">Light</button>
				<button @click="selectTheme('dark')">Dark</button>
				<button @click="selectTheme('fun')">Fun</button>
			</div>
		</div>

		<div class="toolbar-section">
			<span class="label">Spec Profile:</span>
			<div class="buttons">
				<button @click="selectSpec('default')">Default</button>
				<button @click="selectSpec('no-asio')">No ASIO</button>
				<button @click="selectSpec('minimal')">Minimal</button>
			</div>
		</div>

		<div class="toolbar-section">
			<label class="sidebar-toggle">
				<input type="checkbox" :checked="showSidebar" @change="toggleSidebar" />
				<span class="label">Show Sidebar</span>
			</label>
		</div>
	</div>
</template>

<style lang="scss" scoped>
.header-toolbar {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 24px;
	padding: 10px 16px;
	background: #f0f0f0;
	border-bottom: 1px solid #ddd;
	font-family: sans-serif;
	font-size: 13px;

	.toolbar-section {
		display: flex;
		align-items: center;
		gap: 12px;
	}

	.label {
		font-weight: bold;
		color: #555;
		white-space: nowrap;
	}

	.buttons {
		display: flex;
		gap: 8px;
	}

	button {
		padding: 4px 12px;
		border: 1px solid #ccc;
		background: #fff;
		cursor: pointer;
		border-radius: 4px;
		font-size: 11px;
		transition: all 0.2s;

		&:hover {
			border-color: #00abae;
			color: #00abae;
		}

		&:active {
			background: #f0f0f0;
		}
	}

	.sidebar-toggle {
		display: flex;
		align-items: center;
		gap: 8px;
		cursor: pointer;
		user-select: none;

		input {
			cursor: pointer;
		}
	}
}
</style>
