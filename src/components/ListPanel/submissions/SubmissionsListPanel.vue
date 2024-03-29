<template>
	<div class="">
		<list-panel :isSidebarVisible="isSidebarVisible" :items="items" class="">
			<template slot="itemsEmpty">
				<template v-if="isLoading">
					<spinner />
					{{ __('common.loading') }}
				</template>
				<template v-else>
					{{ __('submission.list.empty') }}
				</template>
			</template>

			<template v-slot:item="{item}">
				<table>
					<tbody>
						<slot name="item" :item="item">
							<submissions-list-item
								:key="item.id"
								:item="item"
								:apiUrl="apiUrl"
								:infoUrl="infoUrl"
								:assignParticipantUrl="assignParticipantUrl"
								@addFilter="addFilter"
							/>
						</slot>
					</tbody>
				</table>
			</template>

			<pagination
				v-if="lastPage > 1"
				slot="footer"
				:currentPage="currentPage"
				:isLoading="isLoading"
				:lastPage="lastPage"
				@set-page="setPage"
			/>
		</list-panel>
	</div>
</template>

<script>
import ListPanel from '@/components/ListPanel/ListPanel.vue';
import Pagination from '@/components/Pagination/Pagination.vue';
import SubmissionsListItem from '@/components/ListPanel/submissions/SubmissionsListItem.vue';
import fetch from '@/mixins/fetch';

export default {
	mixins: [fetch],
	components: {
		ListPanel,
		Pagination,
		SubmissionsListItem
	},
	props: {
		addUrl: {
			type: String,
			required: true
		},
		assignParticipantUrl: {
			type: String,
			default() {
				return '';
			}
		},
		filters: {
			type: Array,
			default() {
				return [];
			}
		},
		id: {
			type: String,
			required: true
		},
		infoUrl: {
			type: String,
			required: true
		},
		items: {
			type: Array,
			default() {
				return [];
			}
		},
		itemsMax: {
			type: Number,
			defaut() {
				return 0;
			}
		},
		title: {
			type: String,
			required: true
		},
		allowSubmissions: {
			type: Boolean,
			default() {
				return true;
			}
		}
	},
	data() {
		return {
			isSidebarVisible: false
		};
	},
	computed: {
		/**
		 * Can the current user filter the list?
		 */
		currentUserCanFilter() {
			return this.userHasRole([
				pkp.const.ROLE_ID_MANAGER,
				pkp.const.ROLE_ID_SUB_EDITOR,
				pkp.const.ROLE_ID_ASSISTANT
			]);
		},

		/**
		 * Does the current user have a role which can create a new submission?
		 */
		currentUserCanAddSubmission() {
			return (
				this.allowSubmissions &&
				this.userHasRole([
					pkp.const.ROLE_ID_MANAGER,
					pkp.const.ROLE_ID_SUB_EDITOR,
					pkp.const.ROLE_ID_ASSISTANT,
					pkp.const.ROLE_ID_AUTHOR,
					pkp.const.ROLE_ID_REVIEWER
				])
			);
		}
	},
	methods: {
		/**
		 * Add a filter
		 *
		 * @param {String} param
		 * @param {mixed} value
		 */
		addFilter(param, value) {
			// Don't allow other filters to be active when the
			// isIncomplete filter is active
			if (param === 'isIncomplete') {
				this.activeFilters = {isIncomplete: value};
				this.get();
			} else {
				let newFilters = {...this.activeFilters};
				if (newFilters.hasOwnProperty('isIncomplete')) {
					delete newFilters.isIncomplete;
				}
				if (
					[
						'isOverdue',
						'daysInactive',
						'assignedTo',
						'issueIds',
						'sectionIds',
						'categoryIds',
						'assignedTo'
					].includes(param)
				) {
					newFilters[param] = value;
				} else {
					if (!newFilters[param]) {
						newFilters[param] = [];
					}
					newFilters[param].push(value);
				}
				this.activeFilters = newFilters;
			}
		},

		/**
		 * Is a filter currently active?
		 *
		 * @param {string} param The filter param
		 * @param {mixed} value The filter value
		 * @return {Boolean}
		 */
		isFilterActive(param, value) {
			if (!Object.keys(this.activeFilters).includes(param)) {
				return false;
			} else if (Array.isArray(this.activeFilters[param])) {
				return this.activeFilters[param].includes(value);
			} else if (['isOverdue', 'daysInactive'].includes(param)) {
				return true;
			} else {
				return this.activeFilters[param] === value;
			}
		},

		/**
		 * Remove a filter
		 *
		 * @param {String} param
		 * @param {mixed} value
		 */
		removeFilter(param, value) {
			let newFilters = {...this.activeFilters};
			if (
				[
					'isIncomplete',
					'isOverdue',
					'daysInactive',
					'issueIds',
					'sectionIds',
					'categoryIds',
					'assignedTo'
				].includes(param)
			) {
				delete newFilters[param];
			} else {
				newFilters[param] = newFilters[param].filter(v => v !== value);
			}
			this.activeFilters = newFilters;
		},

		/**
		 * Update the list of items
		 *
		 * @param {Array} items
		 * @param {Number} itemsMax
		 */
		setItems(items, itemsMax) {
			this.$emit('set', this.id, {
				items,
				itemsMax
			});
		},

		/**
		 * Helper function to determine if the current user has a role
		 *
		 * @param int|array roles The role ID to look for (pkp.const.ROLE_ID...)
		 * @return bool
		 */
		userHasRole(roles) {
			if (!Array.isArray(roles)) {
				roles = [roles];
			}

			var hasRole = false;
			roles.forEach(role => {
				if (pkp.currentUser.roles.indexOf(role) > -1) {
					hasRole = true;
				}
			});

			return hasRole;
		}
	},
	mounted() {
		/**
		 * Refresh the list when a submission is updated
		 */
		pkp.eventBus.$on('updated:submission', () => this.get());

		/**
		 * Remove a submission from the list when it is deleted
		 */
		pkp.eventBus.$on('deleted:submission', data => {
			if (
				!data.id ||
				!this.items.find(submission => submission.id === data.id)
			) {
				return;
			}
			this.items = this.items.filter(item => {
				return data.id !== item.id;
			});
			this.itemsMax--;
		});
	},
	destroyed() {
		pkp.eventBus.$off('updated:submission');
		pkp.eventBus.$off('deleted:submission');
	}
};
</script>
