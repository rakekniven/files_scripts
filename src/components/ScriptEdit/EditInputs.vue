<template>
	<div>
		<h3>{{ t('files_scripts', 'User inputs') }}</h3>
		<div class="section-description">
			{{ t('files_scripts', 'Specify any number of input values the user may provide when running this action. These will be accessible to the script via the get_input() function.') }}
		</div>

		<div v-if="loading" class="icon-loading" />
		<div v-else class="script-input">
			<input v-model="inputName"
				type="text"
				class="input-name"
				:placeholder="t('files_scripts', 'Variable name')">
			<input v-model="inputDescription"
				type="text"
				class="input-description"
				:placeholder="t('files_scripts', 'User prompt...')">
			<div class="input-action">
				<Actions>
					<ActionButton @click="addInput()">
						<template #icon>
							<Plus :size="20" />
						</template>
					</ActionButton>
				</Actions>
			</div>
		</div>
		<div v-for="(scriptInput, idx) in scriptInputs"
			:key="idx"
			class="script-input">
			<div class="input-name">
				{{ scriptInput.name }}
			</div>
			<div class="input-description">
				{{ scriptInput.description }}
			</div>
			<div class="input-action">
				<Actions>
					<ActionButton @click="remove(scriptInput.name)">
						<template #icon>
							<Delete :size="20" @click="remove(scriptInput)" />
						</template>
					</ActionButton>
				</Actions>
			</div>
		</div>
	</div>
</template>

<script lang="ts">
import Actions from '@nextcloud/vue/dist/Components/Actions'
import ActionButton from '@nextcloud/vue/dist/Components/ActionButton'
import Plus from 'vue-material-design-icons/Plus.vue'
import Delete from 'vue-material-design-icons/Delete.vue'
import { ScriptInput } from '../../types/script'
import Vue from 'vue'
import { api } from '../../api/script'
import { translate as t } from '../../l10n'

export default {
	name: 'EditInputs',
	components: {
		Actions,
		ActionButton,
		Plus,
		Delete,
	},
	props: {
		scriptId: Number,
	},
	data() {
		return {
			inputName: '',
			inputDescription: '',
			scriptInputs: {},
			loading: true,
		}
	},
	watch: {
		scriptId(newVal) {
			(newVal !== null) && this.fetchInputs()
		},
	},
	mounted() {
		this.fetchInputs()
	},
	methods: {
		t,
		addInput() {
			if (this.inputName.trim().length === 0) {
				return
			}

			this.scriptInputs[this.inputName] = {
				name: this.inputName,
				description: this.inputDescription,
			} as ScriptInput

			this.inputName = ''
			this.inputDescription = ''

			this.updated()
		},
		remove(scriptInput: ScriptInput) {
			Vue.delete(this.scriptInputs, scriptInput.name)
			this.updated()
		},
		updated() {
			this.$emit('changed', Object.values(this.scriptInputs))
		},
		async fetchInputs() {
			if (this.scriptId === null) {
				this.loading = false
				return
			}

			const inputs = await api.getScriptInputs(this.scriptId)
			const scriptInputs = {}
			inputs.forEach(function(input: ScriptInput) {
				scriptInputs[input.name] = input
			})
			this.scriptInputs = scriptInputs
			this.loading = false
		},
	},
}
</script>

<style scoped>
h3 {
	margin-top: 24px;
}
.section-description {
	opacity: .7;
	margin-bottom: 16px;
}
.script-input {
	display: flex;
}
.input-name {
	flex: 0 1 30%;
}
.input-description {
	flex: 1 2 60%;
}
.input-action {
	flex: 0 0 10%;
}
</style>
