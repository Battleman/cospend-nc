<template>
	<NcAppNavigation>
		<template #list>
			<NcAppNavigationItem v-if="!pageIsPublic && !loading"
				class="addProjectItem"
				:editable="true"
				:title="t('cospend', 'New project')"
				:edit-placeholder="t('cospend', 'New project name')"
				:edit-label="t('cospend', 'New empty project')"
				:loading="importingProject"
				:menu-open="importMenuOpen"
				@click="importMenuOpen = true"
				@update:title="$emit('create-project', $event)"
				@update:menuOpen="updateImportMenuOpen">
				<template #icon>
					<PlusIcon :size="20" />
				</template>
				<template #actions>
					<NcActionButton
						:close-after-click="true"
						@click="onImportClick">
						<template #icon>
							<FileImportIcon
								class="icon"
								:size="20" />
						</template>
						{{ t('cospend', 'Import csv project') }}
					</NcActionButton>
					<NcActionButton
						:close-after-click="true"
						@click="onImportSWClick">
						<template #icon>
							<FileImportIcon
								class="icon"
								:size="20" />
						</template>
						{{ t('cospend', 'Import SplitWise project') }}
					</NcActionButton>
				</template>
			</NcAppNavigationItem>
			<h2 v-if="loading"
				class="icon-loading-small loading-icon" />
			<NcEmptyContent v-else-if="sortedProjectIds.length === 0"
				:title="t('cospend', 'No projects yet')">
				<template #icon>
					<FolderIcon :size="20" />
				</template>
			</NcEmptyContent>
			<AppNavigationProjectItem
				v-for="id in sortedProjectIds"
				:key="id"
				:project="projects[id]"
				:members="projects[id].members"
				:selected="id === selectedProjectId"
				:selected-member-id="selectedMemberId"
				:member-order="cospend.memberOrder"
				@project-clicked="onProjectClicked"
				@delete-project="onDeleteProject"
				@stats-clicked="onStatsClicked"
				@settle-clicked="onSettleClicked"
				@detail-clicked="onDetailClicked"
				@share-clicked="onShareClicked"
				@new-member-clicked="onNewMemberClicked"
				@member-edited="onMemberEdited"
				@member-click="$emit('member-click', id, $event)" />
		</template>
		<template #footer>
			<div id="app-settings">
				<div id="app-settings-header">
					<!--button class="settings-button" @click="showSettings">
						{{ t('cospend', 'Cospend settings') }}
					</button-->
					<NcAppNavigationItem
						:title="t('cospend', 'Cospend settings')"
						@click="showSettings">
						<template #icon>
							<CogIcon
								class="icon"
								:size="20" />
						</template>
					</NcAppNavigationItem>
				</div>
			</div>
		</template>
	</NcAppNavigation>
</template>

<script>
import FolderIcon from 'vue-material-design-icons/Folder.vue'
import PlusIcon from 'vue-material-design-icons/Plus.vue'
import FileImportIcon from 'vue-material-design-icons/FileImport.vue'
import CogIcon from 'vue-material-design-icons/Cog.vue'

import NcAppNavigation from '@nextcloud/vue/dist/Components/NcAppNavigation.js'
import NcEmptyContent from '@nextcloud/vue/dist/Components/NcEmptyContent.js'
import NcAppNavigationItem from '@nextcloud/vue/dist/Components/NcAppNavigationItem.js'
import NcActionButton from '@nextcloud/vue/dist/Components/NcActionButton.js'

import AppNavigationProjectItem from './AppNavigationProjectItem.vue'

import cospend from '../state.js'
import * as constants from '../constants.js'
import { strcmp, importCospendProject, importSWProject } from '../utils.js'

import ClickOutside from 'vue-click-outside'
import { emit } from '@nextcloud/event-bus'
import { showSuccess } from '@nextcloud/dialogs'

export default {
	name: 'CospendNavigation',
	components: {
		AppNavigationProjectItem,
		NcAppNavigation,
		NcEmptyContent,
		NcAppNavigationItem,
		NcActionButton,
		CogIcon,
		FileImportIcon,
		PlusIcon,
		FolderIcon,
	},
	directives: {
		ClickOutside,
	},
	props: {
		projects: {
			type: Object,
			required: true,
		},
		selectedProjectId: {
			type: String,
			default: '',
		},
		selectedMemberId: {
			type: Number,
			default: null,
		},
		loading: {
			type: Boolean,
			default: false,
		},
	},
	data() {
		return {
			opened: false,
			creating: false,
			cospend,
			pageIsPublic: cospend.pageIsPublic,
			importMenuOpen: false,
			importingProject: false,
		}
	},
	computed: {
		sortedProjectIds() {
			if (this.cospend.sortOrder === 'name') {
				return Object.keys(this.projects).sort((a, b) => {
					return strcmp(this.projects[a].name, this.projects[b].name)
				})
			} else if (this.cospend.sortOrder === 'change') {
				return Object.keys(this.projects).sort((a, b) => {
					return this.projects[b].lastchanged - this.projects[a].lastchanged
				})
			} else {
				return Object.keys(this.projects)
			}
		},
		editionAccess() {
			return this.selectedProjectId && this.projects[this.selectedProjectId].myaccesslevel >= constants.ACCESS.PARTICIPANT
		},
	},
	beforeMount() {
	},
	methods: {
		showSettings() {
			emit('show-settings')
		},
		toggleMenu() {
			this.opened = !this.opened
		},
		closeMenu() {
			this.opened = false
		},
		onProjectClicked(projectid) {
			this.$emit('project-clicked', projectid)
		},
		onDeleteProject(projectid) {
			this.$emit('delete-project', projectid)
		},
		onStatsClicked(projectid) {
			this.$emit('stats-clicked', projectid)
		},
		onSettleClicked(projectid) {
			this.$emit('settle-clicked', projectid)
		},
		onDetailClicked(projectid) {
			this.$emit('detail-clicked', projectid)
		},
		onShareClicked(projectid) {
			this.$emit('share-clicked', projectid)
		},
		onNewMemberClicked(projectid) {
			this.$emit('new-member-clicked', projectid)
		},
		onMemberEdited(projectid, memberid) {
			this.$emit('member-edited', projectid, memberid)
		},
		onImportClick() {
			importCospendProject(() => {
				this.importingProject = true
			}, (data) => {
				this.$emit('project-imported', data)
				showSuccess(t('cospend', 'Project imported'))
			}, () => {
				this.importingProject = false
			})
		},
		onImportSWClick() {
			importSWProject(() => {
				this.importingProject = true
			}, (data) => {
				this.$emit('project-imported', data)
				showSuccess(t('cospend', 'Project imported'))
			}, () => {
				this.importingProject = false
			})
		},
		updateImportMenuOpen(isOpen) {
			if (!isOpen) {
				this.importMenuOpen = false
			}
		},
	},
}
</script>
<style scoped lang="scss">
.addProjectItem {
	position: sticky;
	top: 0;
	z-index: 1000;
	border-bottom: 1px solid var(--color-border);
	::v-deep .app-navigation-entry {
		background-color: var(--color-main-background-blur, var(--color-main-background));
		backdrop-filter: var(--filter-background-blur, none);
		&:hover {
			background-color: var(--color-background-hover);
		}
	}
}

::v-deep .selectedproject,
::v-deep .selectedmember {
	> .app-navigation-entry {
		background: var(--color-primary-light, lightgrey);
	}

	> .app-navigation-entry a {
		font-weight: bold;
	}
}

::v-deep .selectedmember .app-navigation-entry__title {
	text-decoration: underline;
}

#app-settings-content {
	p {
		margin-top: 20px;
		margin-bottom: 20px;
		color: var(--color-text-light);
	}
}

.project-create {
	order: 1;
	display: flex;
	height: 44px;
	form {
		display: flex;
		flex-grow: 1;
		input[type='text'] {
			flex-grow: 1;
		}
	}
}

.buttonItem {
	border-bottom: solid 1px var(--color-border);
}

.loading-icon {
	margin-top: 16px;
}
</style>
