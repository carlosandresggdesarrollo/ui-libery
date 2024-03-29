<template>
	<div class="">
		<link
			href="http://148.202.34.240:8008/app/seuomp/udg_resources/css/tablas.css"
			rel="stylesheet"
		/>
		<list-panel :items="items" class="listPanel--submissionFiles">
			<pkp-header slot="header">
				<h2>{{ title }}</h2>
				<template slot="actions">
					<div class="row g-0">
						<div class="col-9"></div>
						<div
							style="display:inline-block;margin:auto; text-align:right;"
							class="col-2"
						>
							<a ref="addFileButton" @click="openFileBrowser">
								<img
									class="img-fluid"
									style="width:20%;height:40%;display:block;margin-left:110px;"
									src="http://148.202.34.240:8008/app/seuomp/udg/AñadirSimple.png"
								/>
							</a>
						</div>
						<div class="col-1">
							<p
								style="display:block;margin:auto;font-weight:bold;color:#174F77"
							>
								Archivo
							</p>
						</div>
					</div>
					<br />
					<div class="row">
						<div class="col-sm-12">
							<table class="table">
								<thead>
									<tr>
										<th colspan="2" style="text-align:center;">ARCHIVO</th>
										<th style="border-left: #fff 1px solid; text-align:center;">
											TIPO DE OBRA *
										</th>
									</tr>
								</thead>
								<tbody></tbody>
							</table>
						</div>
					</div>
				</template>
			</pkp-header>

			<template slot="itemsEmpty">
				{{ emptyLabel }}
			</template>

			<template v-slot:item="{item}">
				<table>
					<tbody>
						<slot name="" :item="item">
							<submission-files-list-item
								:apiUrl="apiUrl"
								:cancelUploadLabel="cancelUploadLabel"
								:genrePromptLabel="genrePromptLabel"
								:fileStage="fileStage"
								:genres="genres"
								:item="item"
								:otherLabel="otherLabel"
								:stageId="stageId"
								@cancel="cancelUpload"
								@edit="edit"
								@remove="remove"
								@update="updateItem"
							/>
						</slot>
					</tbody>
				</table>
			</template>
		</list-panel>
		<file-uploader
			ref="uploader"
			:apiUrl="apiUrl"
			:filenameLocale="primaryLocale"
			:files="items"
			:id="id + '-uploader'"
			:options="options"
			:queryParams="{fileStage}"
			:uploadProgressLabel="uploadProgressLabel"
			@updated:files="setFiles"
		/>
		<modal :closeLabel="__('common.close')" name="form" :title="editingLabel">
			<pkp-form v-bind="activeForm" @set="setForm" @success="formSuccess" />
		</modal>
	</div>
</template>

<script>
import FileUploader from '@/components/FileUploader/FileUploader.vue';
import ListPanel from '@/components/ListPanel/ListPanel.vue';
import Modal from '@/components/Modal/Modal.vue';
import PkpForm from '@/components/Form/Form.vue';
import PkpHeader from '@/components/Header/Header.vue';
import SubmissionFilesListItem from '@/components/ListPanel/submissionFiles/SubmissionFilesListItem.vue';
import cloneDeep from 'clone-deep';

export default {
	components: {
		FileUploader,
		ListPanel,
		Modal,
		PkpForm,
		PkpHeader,
		SubmissionFilesListItem
	},
	props: {
		addFileLabel: {
			type: String,
			required: true
		},
		apiUrl: {
			type: String,
			required: true
		},
		cancelUploadLabel: {
			type: String,
			required: true
		},
		emptyLabel: {
			type: String,
			required: true
		},
		emptyAddLabel: {
			type: String,
			required: true
		},
		fileStage: {
			type: Number,
			required: true
		},
		form: {
			type: Object
		},
		genrePromptLabel: {
			type: String,
			required: true
		},
		genres: {
			type: Array,
			required: true
		},
		id: {
			type: String,
			required: true
		},
		items: {
			type: Array,
			default() {
				return [];
			}
		},
		options: {
			type: Object,
			default() {
				return {};
			}
		},
		otherLabel: {
			type: String,
			required: true
		},
		primaryLocale: {
			type: String,
			required: true
		},
		removeConfirmLabel: {
			type: String,
			required: true
		},
		stageId: {
			type: Number,
			required: true
		},
		title: {
			type: String,
			required: true
		},
		uploadProgressLabel: {
			type: String,
			required: true
		}
	},
	data() {
		return {
			activeForm: {},
			dragEventCounter: 0,
			editingLabel: '',
			isDragging: false,
			status: ''
		};
	},
	methods: {
		/**
		 * Cancel an upload in progress or completed but not yet
		 * saved as a submission file
		 *
		 * This will not remove a file once it has been uploaded.
		 * Use this.remove(item).
		 */
		cancelUpload(id) {
			this.$refs.uploader.cancelUpload(id);
		},

		/**
		 * Open the editing panel for a file
		 *
		 * @param {Object} item The item to edit
		 */
		edit(item) {
			let activeForm = {...cloneDeep(this.form)};
			activeForm.action =
				activeForm.action + '/' + item.id + '?stageId=' + this.stageId;
			// Set values that match the form fields
			activeForm.fields = activeForm.fields.map(field => {
				if (item[field.name] != null) {
					field.value = item[field.name];
				}
				return field;
			});
			this.activeForm = activeForm;
			this.editingLabel = this.__('common.editItem', {
				name: this.localize(item.name)
			});
			this.$modal.show('form');
		},

		/**
		 * Fired when the edit form has been saved
		 *
		 * @param {Object} item The item to update
		 */
		formSuccess(item) {
			this.updateItem(item);
			this.$modal.hide('form');
			this.activeForm = {};
			this.$el.querySelector('#edit-' + item.id).focus();
		},

		/**
		 * Open the file browser dialog
		 */
		openFileBrowser() {
			this.$refs.uploader.openFileBrowser();
		},

		/**
		 * Remove a file
		 *
		 * @param {Object} item The file in this.items to remove
		 */
		remove(item) {
			this.openDialog({
				name: 'remove',
				title: this.__('common.remove'),
				message: this.removeConfirmLabel,
				actions: [
					{
						label: this.__('common.yes'),
						isPrimary: true,
						callback: () => {
							var self = this;
							$.ajax({
								url: this.apiUrl + '/' + item.id + '?stageId=' + this.stageId,
								type: 'POST',
								headers: {
									'X-Csrf-Token': pkp.currentUser.csrfToken,
									'X-Http-Method-Override': 'DELETE'
								},
								error: self.ajaxErrorCallback,
								success(r) {
									const items = self.items.filter(item => item.id !== r.id);
									self.$emit('set', self.id, {items});
									self.$modal.hide('remove');
									self.$refs.addFileButton.$el.focus();
								}
							});
						}
					},
					{
						label: this.__('common.no'),
						isWarnable: true,
						callback: () => this.$modal.hide('remove')
					}
				]
			});
		},

		/**
		 * Set the files
		 */
		setFiles(files) {
			this.$emit('set', this.id, {items: files});
		},

		/**
		 * Update the data in the active form
		 *
		 * @param {Number} key The id of the form
		 * @param {Object} data Key/value map of the data to be changed
		 */
		setForm(key, data) {
			let activeForm = {...this.activeForm};
			Object.keys(data).forEach(function(key) {
				activeForm[key] = data[key];
			});
			this.activeForm = activeForm;
		},

		/**
		 * Update one of the items in the list
		 *
		 * @param {Object} newItem The item to update
		 */
		updateItem(newItem) {
			const items = this.items.map(item => {
				return item.id === newItem.id ? newItem : item;
			});
			this.$emit('set', this.id, {items});
		}
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';

// Restrict the drag-and-drop area to the panel
.submissionFilesListPanel {
	position: relative;
}
</style>
