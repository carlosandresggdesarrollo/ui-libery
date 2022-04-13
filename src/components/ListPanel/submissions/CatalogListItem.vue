<template>
	<div class="row" :class="{'': isFeatured}" style="text-align:justify;">
		<div class="col-sm-1 text-center">
			<div class="">
				<div class="">
					{{ item.id }}
				</div>
			</div>
		</div>
		<div class="col-sm-3 " style="justify-content:center;">
			<div>
				<div class="">
					<span v-if="currentPublication.authorsStringShort" class="">
						{{ currentPublication.authorsStringShort }}
					</span>
				</div>
				<div class="">
					{{ localize(currentPublication.fullTitle) }}
				</div>
			</div>
		</div>

		<div class="col-sm-3">
			<pkp-button
				element="a"
				:href="item.urlWorkflow"
				style="display:flex;justify-content:center;"
			>
				{{ __('submission.list.viewSubmission') }}
			</pkp-button>
		</div>
		<div class="col-sm-2" style="justify-content:center;">
			<pkp-button
				element="a"
				:href="item.urlPublished"
				style="display:flex;justify-content:center;"
			>
				{{ __('submission.list.viewEntry') }}
			</pkp-button>
		</div>
		<div class="col-sm-1" style="justify-content:center;">
			<button class="form-control" @click.prevent="toggleFeatured">
				<icon v-if="isFeatured" icon="check-square-o" />
				<icon v-else icon="square-o" />
				<span class="-screenReader">
					<template v-if="isFeatured">
						{{ __('catalog.manage.isFeatured') }}
					</template>
					<template v-else>
						{{ __('catalog.manage.isNotFeatured') }}
					</template>
				</span>
			</button>
		</div>
		<div class="col-sm-1" style="justify-content: center;">
			<button class="form-control" @click.prevent="toggleNewRelease">
				<icon :icon="isNewRelease ? 'check-square-o' : 'square-o'" />
				<span class="-screenReader">
					<template v-if="isNewRelease">
						{{ __('catalog.manage.isNewRelease') }}
					</template>
					<template v-else>
						{{ __('catalog.manage.isNotNewRelease') }}
					</template>
				</span>
			</button>
		</div>
		<div class="col-sm-1"></div>
		<orderer
			v-if="isOrdering"
			@up="$emit('order-up', item)"
			@down="$emit('order-down', item)"
			:itemId="item.id"
			:itemTitle="item.title"
		/>
		<div
			v-if="isSaving || isLoading"
			class="listPanel__itemMask--catalog"
		></div>
	</div>
</template>

<script>
import Orderer from '@/components/Orderer/Orderer.vue';
import ajaxError from '@/mixins/ajaxError';

export default {
	name: 'CatalogListItem',
	components: {
		Orderer
	},
	mixins: [ajaxError],
	props: [
		'apiUrl',
		'filterAssocType',
		'filterAssocId',
		'isLoading',
		'isOrdering',
		'item'
	],
	data() {
		return {
			isSaving: false
		};
	},
	computed: {
		/**
		 * The current publication of the submission
		 *
		 * @return {Object}
		 */
		currentPublication() {
			return this.item.publications.find(
				publication => publication.id === this.item.currentPublicationId
			);
		},

		/**
		 * Is the submission featured in the current filtered view?
		 * press, category or series
		 *
		 * @return {Boolean}
		 */
		isFeatured() {
			if (!this.item.hasOwnProperty('featured')) {
				return false;
			}
			const feature = this.item.featured.find(feature => {
				return (
					feature.assoc_type === this.filterAssocType &&
					feature.assoc_id === this.filterAssocId
				);
			});
			return typeof feature !== 'undefined';
		},

		/**
		 * Is the submission a new release in the current filtered view?
		 * press, category or series
		 *
		 * @return {Boolean}
		 */
		isNewRelease() {
			if (!this.item.hasOwnProperty('newRelease')) {
				return false;
			}
			const newRelease = this.item.newRelease.find(newRelease => {
				return (
					newRelease.assoc_type === this.filterAssocType &&
					newRelease.assoc_id === this.filterAssocId
				);
			});
			return typeof newRelease !== 'undefined';
		}
	},
	methods: {
		/**
		 * Toggle the checkbox when clicked
		 */
		toggleFeatured() {
			const isFeatured = this.item.featured.find(feature => {
				return feature.assoc_type === this.filterAssocType;
			});
			if (isFeatured) {
				this.item.featured = this.item.featured.filter(feature => {
					return feature.assoc_type !== this.filterAssocType;
				});
			} else {
				this.item.featured.push({
					assoc_type: this.filterAssocType,
					assoc_id: this.filterAssocId,
					seq: 1
				});
			}
			this.saveDisplayFlags();
		},

		/**
		 * Toggle the checkbox when clicked
		 */
		toggleNewRelease() {
			const matchingNewReleases = this.item.newRelease.find(newRelease => {
				return newRelease.assoc_type === this.filterAssocType;
			});
			const isNewRelease = typeof matchingNewReleases !== 'undefined';
			if (isNewRelease) {
				this.item.newRelease = this.item.newRelease.filter(newRelease => {
					return newRelease.assoc_type !== this.filterAssocType;
				});
			} else {
				this.item.newRelease.push({
					assoc_type: this.filterAssocType,
					assoc_id: this.filterAssocId,
					seq: 1
				});
			}
			this.saveDisplayFlags();
		},

		/**
		 * Post updates to the featured or new release status of a submission
		 */
		saveDisplayFlags() {
			this.isLoading = true;

			const self = this;
			$.ajax({
				url: this.apiUrl + '/saveDisplayFlags',
				type: 'POST',
				headers: {
					'X-Csrf-Token': pkp.currentUser.csrfToken
				},
				data: {
					submissionId: this.item.id,
					featured: this.item.featured,
					newRelease: this.item.newRelease
				},
				error: function(r) {
					self.ajaxErrorCallback(r);
				},
				success: function(r) {
					if (typeof r.featured !== 'undefined') {
						self.$emit('update:item', {...self.item, featured: r.featured});
					}
					if (typeof r.newRelease !== 'undefined') {
						self.$emit('update:item', {...self.item, newRelease: r.newRelease});
					}
				},
				complete() {
					self.isLoading = false;
				}
			});
		}
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';

// Shift padding to the catalog item so that focus indicator
// takes up full height
.listPanel--catalog .listPanel__item {
	position: relative;
	padding: 0;
}

.listPanel__itemIdentity--catalog {
	padding: 0.75rem 0.5rem 0.75rem 3.5rem;
}

.listPanel__item--catalog__id {
	position: absolute;
	top: 0.75rem;
	left: 1rem;
	font-size: @font-tiny;
	line-height: 22px; // Match baseline of title
	color: @text;
}

.listPanel__itemActions--catalog {
	align-self: stretch;
}

.listPanel__item--catalog__select {
	align-self: stretch;
	width: 6rem;
	margin-left: 0;
	background: transparent;
	border: 1px solid transparent;
	border-left: @grid-border;

	&:focus {
		outline: 0;
		border-color: @primary;
	}

	+ .listPanel__item--catalog__select {
		right: 0;
	}
}

.listPanel__item--catalog__select--first {
	margin-left: 0.5rem;
}

.listPanel__item--catalog {
	.orderer__dragDrop,
	.orderer__up,
	.orderer__down {
		width: 4em;
	}

	.orderer__up {
		right: 4em;
	}
}

.listPanel__itemMask--catalog {
	position: absolute;
	top: 0;
	bottom: 0;
	left: 0;
	right: 0;
	background: #fff;
	opacity: 0.5;
}
</style>
