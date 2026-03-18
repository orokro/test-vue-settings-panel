<script setup>
import { reactive, ref, computed } from 'vue'
import { VueSettingsPanel, TYPES, createSettings } from 'vue-settings-panel'
import 'vue-settings-panel/dist/style.css'
import ThemeSwitcher from './components/ThemeSwitcher.vue'

// Import custom components
import Vector2Edit from './components/Vector2Edit.vue'
import Vector3Edit from './components/Vector3Edit.vue'
import PanEdit from './components/PanEdit.vue'

// Define custom types
const CUSTOM_TYPES = {
  Vector2: {
    name: 'Vector2',
    slug: 'vector2',
    defaultValue: { x: 0, y: 0 },
    component: Vector2Edit,
  },
  Vector3: {
    name: 'Vector3',
    slug: 'vector3',
    defaultValue: { x: 0, y: 0, z: 0 },
    component: Vector3Edit,
  },
  Pan: {
    name: 'Pan',
    slug: 'pan',
    defaultValue: 0,
    component: PanEdit,
  }
}

// 1. Define Specification Profiles
const specs = {
  default: {
    sidePanel: { enabled: true, autoCollapse: true },
    search: { nonMatchedCategories: 'gray', nonMatchSettings: 'collapse' },
    categories: [
      { name: "Audio Engine", slug: "audio", desc: "Configure core audio processing", icon: 'speaker', categories: [
        { name: "Device Settings", slug: "device" },
        { name: "Buffer & Latency", slug: "buffer" }
      ]},
      { name: "Control Surfaces", slug: "control", desc: "Connect MIDI controllers", icon: 'settings_input_component', categories: [
        { name: "MIDI Mapping", slug: "midi" },
        { name: "Hardware Integration", slug: "hw" }
      ]},
      { name: "Interface", slug: "ui", desc: "Customize DAW appearance", icon: 'palette', categories: [
        { name: "General UI", slug: "general" },
        { name: "Themes", slug: "themes" }
      ]},
      { name: "Python Scripting", slug: "scripting", desc: "Automation and extension", icon: 'code', categories: [
        { name: "Editor Options", slug: "editor" },
        { name: "Runtime", slug: "runtime" }
      ]},
      { name: "Advanced", slug: "advanced", desc: "Experimental features", icon: 'https://vuejs.org/images/logo.png' }
    ],
    settings: {
      driverType: { name: 'Driver Type', desc: 'Select audio driver', cats: ['audio.device'], type: TYPES.Select, opts: { options: ["ASIO", "CoreAudio", "WASAPI", "DirectSound"] }, default: "ASIO" },
      sampleRate: { name: 'Sample Rate', desc: 'Audio resolution', cats: ['audio.device'], type: TYPES.Select, opts: { options: ["44100 Hz", "48000 Hz", "88200 Hz", "96000 Hz"] }, default: "48000 Hz" },
      bufferSize: { name: 'Buffer Size', desc: 'Latency control', cats: ['audio.buffer'], tags: ['latency', 'delay'], type: TYPES.Number, opts: { min: 32, max: 4096, step: 32 }, default: 512 },
      multiCore: { name: 'Multi-Core Processing', desc: 'Parallel processing', cats: ['audio.buffer'], type: TYPES.Boolean, default: true },
      enableMidi: { name: 'Enable MIDI Input', desc: 'Allow MIDI data', cats: ['control.midi'], type: TYPES.Boolean, default: true },
      midiClock: { name: 'Send MIDI Clock', desc: 'Sync hardware', cats: ['control.midi'], type: TYPES.Boolean, default: false, show: (s) => s.enableMidi },
      surfaceType: { name: 'Surface Protocol', desc: 'HW protocol', cats: ['control.hw'], type: TYPES.Select, opts: { options: ["Native", "Mackie Control", "HUI", "OSC"] }, default: "Native" },
      autoSave: { name: 'Auto Save', desc: 'Background saving', cats: ['ui.general'], type: TYPES.Boolean, default: true },
      saveInterval: { name: 'Save Interval', desc: 'Minutes between saves', cats: ['ui.general'], type: TYPES.FloatRange, opts: { min: 1, max: 60, step: 1 }, default: 10, show: (s) => s.autoSave },
      accentColor: { name: 'Accent Color', desc: 'UI highlight color', cats: ['ui.themes'], type: TYPES.Color, default: '#00abae' },
      uiScale: { name: 'UI Scaling', desc: 'Adjust UI size', cats: ['ui.themes'], type: TYPES.FloatRange, opts: { min: 0.5, max: 2, step: 0.1 }, default: 1 },
      autoComplete: { name: 'Auto-Completion', desc: 'Python suggestions', cats: ['scripting.editor'], type: TYPES.Boolean, default: true },
      fontSize: { name: 'Editor Font Size', desc: 'Script text size', cats: ['scripting.editor'], type: TYPES.Number, opts: { min: 8, max: 32, step: 1 }, default: 14, mount: 'bottom' },
      safeMode: { name: 'Scripts Safe Mode', desc: 'FS protection', cats: ['scripting.runtime'], type: TYPES.Boolean, default: true },
      enableBeta: { name: 'Beta Features', desc: 'Experimental algorithms', cats: ['advanced'], tags: ['experimental', 'test'], type: TYPES.Boolean, default: false },
      masterGain: { name: 'Master Output Gain', desc: 'Calibration level for the main output stage', cats: ['advanced'], tags: ['gain', 'volume', 'loudness'], type: TYPES.FloatRange, opts: { min: -60, max: 12, step: 0.1, showInput: true }, default: 0 },
      cameraPos: { name: 'Camera Position', desc: '3D viewport camera', cats: ['advanced'], type: CUSTOM_TYPES.Vector3, default: { x: 0, y: 5, z: -10 } },
      lightDir: { name: 'Light Direction', desc: 'Main light source orientation', cats: ['advanced'], type: CUSTOM_TYPES.Vector2, default: { x: 0.5, y: -0.5 } },
      masterPan: { name: 'Master Pan', desc: 'Stereo balance', cats: ['advanced'], type: CUSTOM_TYPES.Pan, default: 0 },
      userHandle: { 
        name: 'User Handle', 
        desc: 'Unique identifier for community features', 
        cats: ['advanced'], 
        type: TYPES.String, 
        default: 'user_123',
        lint: (val) => val.toLowerCase().replace(/\s+/g, '_'),
        validate: (val) => val.length >= 3 || "Handle must be at least 3 characters"
      },
      userBio: {
        name: 'User Bio',
        desc: 'Tell the community about yourself',
        cats: ['advanced'],
        type: TYPES.String,
        opts: { multiline: true, rows: 3 },
        default: ''
      },
      preferredTheme: {
        name: 'UI Theme Style',
        desc: 'Select your preferred visual style',
        cats: ['ui.themes'],
        type: TYPES.Select,
        opts: { options: ["Classic", "Modern", "High Contrast"], radio: true },
        default: "Modern"
      },
      licenseExpiry: {
        name: 'License Expiry',
        desc: 'Check when your subscription expires',
        cats: ['advanced'],
        type: TYPES.Date,
        default: '2026-12-31'
      },
      dailyBackupTime: {
        name: 'Backup Time',
        desc: 'Time of day to run automated backups',
        cats: ['ui.general'],
        type: TYPES.Time,
        default: '03:00'
      },
      exportShortcut: {
        name: 'Export Shortcut',
        desc: 'Keyboard shortcut to export current project',
        cats: ['ui.general'],
        type: TYPES.KeyInput,
        default: { ctrl: true, shift: false, alt: false, meta: false, key: 'e' }
      },
      customMidiDevice: {
        name: 'External Controller',
        desc: 'Click to capture input from your connected device',
        cats: ['control.hw'],
        type: TYPES.GenericInput,
        opts: { 
          onInputRequest: (set) => {
            setTimeout(() => {
              set("MIDI Dev " + Math.floor(Math.random() * 100))
            }, 1000)
          },
          placeholder: 'Waiting for MIDI...'
        },
        default: ''
      }
    }
  },
  'no-asio': {
    sidePanel: { enabled: true, autoCollapse: true },
    search: { nonMatchedCategories: 'gray', nonMatchSettings: 'collapse' },
    categories: [
      { name: "Audio Engine", slug: "audio", desc: "Configure core audio processing", icon: 'speaker', categories: [
        { name: "Device Settings", slug: "device" }
      ]}
    ],
    settings: {
      driverType: { name: 'Driver Type', desc: 'Select audio driver (ASIO MISSING)', cats: ['audio.device'], type: TYPES.Select, opts: { options: ["CoreAudio", "WASAPI", "DirectSound"] }, default: "CoreAudio" },
      sampleRate: { name: 'Sample Rate', desc: 'Audio resolution', cats: ['audio.device'], type: TYPES.Select, opts: { options: ["44100 Hz", "48000 Hz", "88200 Hz", "96000 Hz"] }, default: "48000 Hz" }
    }
  },
  minimal: {
    sidePanel: { enabled: false, autoCollapse: true },
    search: { nonMatchedCategories: 'collapse', nonMatchSettings: 'collapse' },
    categories: [
      { name: "Essential Settings", slug: "essential", desc: "Crucial parameters only", icon: 'star' }
    ],
    settings: {
      autoSave: { name: 'Auto Save', desc: 'Keep your work safe', cats: ['essential'], type: TYPES.Boolean, default: true },
      themeColor: { name: 'Primary Color', desc: 'Pick your vibe', cats: ['essential'], type: TYPES.Color, default: '#ff4081' },
      masterVolume: { name: 'Volume', desc: 'Master output', cats: ['essential'], tags: ['gain'], type: TYPES.FloatRange, opts: { min: 0, max: 100, step: 1 }, default: 80 }
    }
  }
}

// 2. Reactive State
const activeSpecName = ref('default')
const activeTheme = ref({})
const showSidebarGlobal = ref(true)
const isReactivityTest = ref(window.location.hash === '#reactivity-test')

// Simple hash routing
window.addEventListener('hashchange', () => {
  isReactivityTest.value = window.location.hash === '#reactivity-test'
})

// Using the new createSettings helper
const settingsState = createSettings(specs.default, {
  driverType: "ASIO",
  sampleRate: "48000 Hz",
  bufferSize: 256,
  multiCore: true,
  enableMidi: true,
  midiClock: true,
  surfaceType: "Mackie Control",
  autoSave: true,
  saveInterval: 5,
  accentColor: '#00abae',
  uiScale: 1.1,
  autoComplete: true,
  fontSize: 16,
  safeMode: true,
  enableBeta: false,
  masterGain: -3.5,
  cameraPos: { x: 10, y: 15.5, z: -20 },
  lightDir: { x: 0.1, y: 0.9 },
  masterPan: -12,
  userHandle: 'pro_editor',
  userBio: 'Music producer and sound designer.',
  preferredTheme: 'Modern',
  licenseExpiry: '2026-12-31',
  dailyBackupTime: '03:00',
  exportShortcut: { ctrl: true, shift: false, alt: false, meta: false, key: 'e' },
  customMidiDevice: '',
  themeColor: '#ff4081',
  masterVolume: 80
})

const currentSpec = computed(() => {
  const spec = { ...specs[activeSpecName.value] }
  spec.sidePanel = { ...spec.sidePanel, enabled: showSidebarGlobal.value }
  return spec
})

const handleSettingsChanged = (newSettings) => {
  // console.log('Settings updated:', newSettings)
}

const updateShowSidebar = (val) => {
  showSidebarGlobal.value = val
}

const updateSpec = (name) => {
  activeSpecName.value = name
  // When switching to minimal, we might want to default sidebar to off
  if (name === 'minimal') {
    showSidebarGlobal.value = false
  } else {
    showSidebarGlobal.value = true
  }
}

const toggleReactivityTest = () => {
  window.location.hash = isReactivityTest.value ? '' : '#reactivity-test'
}
</script>

<template>
  <div class="app-wrapper">
    <div class="header-actions">
      <ThemeSwitcher 
        v-model:theme="activeTheme" 
        :currentTheme="activeTheme"
        :showSidebar="showSidebarGlobal" 
        @update:showSidebar="updateShowSidebar"
        @update:spec="updateSpec"
      />
      <button @click="toggleReactivityTest" class="reactivity-toggle">
        {{ isReactivityTest ? 'Exit Reactivity Test' : 'Test Reactivity (Side-by-Side)' }}
      </button>
    </div>
    
    <div class="app-container" :class="{ 'reactivity-test': isReactivityTest }">
      <VueSettingsPanel
        :settings="settingsState"
        :specification="currentSpec"
        :themeColors="activeTheme"
        @settings-changed="handleSettingsChanged"
      />
      <VueSettingsPanel
        v-if="isReactivityTest"
        :settings="settingsState"
        :specification="currentSpec"
        :themeColors="activeTheme"
        @settings-changed="handleSettingsChanged"
      />
    </div>
  </div>
</template>

<style lang="scss">
body, html {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  background-color: #222;
  color: #eee;
  overflow: hidden;
}

#app {
  height: 100vh;
  width: 100vw;
  display: flex;
}

.header-actions {
  display: flex;
  align-items: stretch;
  background: #f0f0f0;
  border-bottom: 1px solid #ddd;
}

.reactivity-toggle {
  background: #00abae;
  color: white;
  border: none;
  padding: 0 20px;
  font-size: 11px;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.2s;
  text-transform: uppercase;
  letter-spacing: 0.05em;

  &:hover {
    background: #008f91;
  }
}

.app-wrapper {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
}

.app-container {
  flex: 1;
  width: 1000px;
  max-width: 95vw;
  height: 700px;
  max-height: 80vh;
  margin: auto;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5);
  border-radius: 12px;
  overflow: hidden;
  background: #111;
  transition: all 0.3s ease;

  &.reactivity-test {
    width: 100%;
    max-width: 100vw;
    height: 100%;
    max-height: 100vh;
    margin: 0;
    border-radius: 0;
    display: flex;
    gap: 1px;
    background: #444; /* Divider */

    & > * {
      flex: 1;
    }
  }
}
</style>
