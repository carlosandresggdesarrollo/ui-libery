<template>
	<fieldset class="" :class="classes">
		<br>
		<legend class="título_nivel_2">
			<template v-if="localeLabel">
				<span class="aria-hidden">{{ localeLabel }}</span>
				<span class="-screenReader">{{ multilingualLabel }}</span>
			</template>
			<template v-else>
				<div class="título_nivel_2">{{ label }}</div>
			</template>
			<span v-if="isRequired" class="pkpFormFieldLabel__required">
				<div>* Campo obligatorio</div>
			</span>
		
			
		</legend>
		<div
			v-if="isPrimaryLocale && description"
			style="margin-right:30px;"
			v-html="description"
			:id="describedByDescriptionId"
		/>
		<field-error
			v-if="errors && errors.length"
			:id="describedByErrorId"
			:messages="errors"
		/>
		<input type="hidden" v-model="selectedValue" />
			<br>
		<div style="margin-right:30px;">
			<label
				v-for="option in localizedOptions"
				:key="option.value"
				class="parrafo_texto" 
				style="margin-right:30px;"
			><!-- pkpFormField--options__legend-->
				<template v-if="!option.isInput">
					<input
						class="form-check-input"
						v-model="selectedValue"
						:value="option.value"
						type="radio"
						:name="localizedName"
						:aria-describedby="describedByIds"
						:aria-invalid="errors && errors.length"
						:disabled="option.disabled"
					/>
					{{ option.label }}	<br>
				</template>
				<template v-else>
					<input
						class="form-check-input "
						type="radio"
						ref="inputRadio"
						:name="localizedName"
						:aria-describedby="describedByIds"
						:aria-invalid="errors && errors.length"
						:disabled="option.disabled"
						@change="selectInput"
						style="margin-right:30px;"
					/>
					<span v-if="option.label">{{ option.label }}</span>
					<input
						class="pkpFormField__input pkpFormField--options__input--text"
						type="text"
						v-model="inputValue"
						ref="inputText"
						:aria-describedby="describedByIds"
						:aria-invalid="errors && errors.length"
						:disabled="option.disabled"
						@focus="selectInput"
					/>
				</template>
					<br>
			</label>
			<multilingual-progress
				v-if="isMultilingual && locales.length > 1"
				:id="multilingualProgressId"
				:count="multilingualFieldsCompleted"
				:total="locales.length"
			/>
			<br>
		</div>
	</fieldset>
</template>

<script>
import FieldOptions from './FieldOptions.vue';

export default {
	name: 'FieldRadioInput',
	extends: FieldOptions,
	props: {
		value: {
			type: String,
			required: true
		}
	},
	data() {
		return {
			inputValue: ''
		};
	},
	computed: {
		/**
		 * Is the input option the currently selected option?
		 *
		 * @return {Boolean}
		 */
		isInputSelected() {
			return !this.localizedOptions
				.filter(opt => !opt.isInput)
				.map(opt => opt.value)
				.includes(this.selectedValue);
		}
	},
	methods: {
		/**
		 * Select the option with an input field
		 */
		selectInput() {
			this.selectedValue = this.inputValue;
			this.$refs.inputRadio[0].checked = true;
			this.$refs.inputText[0].focus();
		},

		/**
		 * Set the seletected value to the input field's vale
		 */
		setInputToSelected() {
			this.selectedValue = this.inputValue;
		}
	},
	watch: {
		/**
		 * When the input value changes, update the selected value if the input
		 * option is the currently selected option
		 */
		inputValue: function(newVal, oldVal) {
			if (newVal === oldVal || !this.isInputSelected) {
				return;
			}
			this.selectedValue = this.inputValue;
		}
	},
	mounted() {
		/**
		 * Put the value into the input field if it doesn't match one of the
		 * existing options.
		 */
		if (this.isInputSelected) {
			this.inputValue = this.isMultilingual
				? this.value[this.localeKey]
				: this.value;
			this.$refs.inputRadio[0].checked = true;
		}
	}
};
</script>

<style lang="less">
@import '../../../styles/_import';

.pkpFormField--options__input--text {
	display: inline-block;
	margin-left: 0.5em;
	padding: 0 0.5em;
	line-height: 1.8rem;
	height: 1.8rem;
}
	.título_principal  {
				font-family:cambersb;
				font-size:34pt;
				color:#174f77;
			}
			.título_nivel_1 {
				font-family:cambersb;
				font-size:27pt;
				color:#445f77;
			}
			.título_nivel_2 {
				font-family:cambersb;
				font-size:18pt;
				color:#737373;
			}
			.título_nivel_3 {
				font-family:cambersb;
				font-size:16pt;
				color:#737373;
			}
			.aclaraciones_co {
				font-family:camberuli;
				font-size:8;
				color:#ED0000;
			}
			.aclaraciones {
				font-family:camberuli;
				font-size:8;
				color:#ED0000;
			}
			.campos_tablas {
				font-family:camberlg;
				font-size:16pt;
				color:#fff;
			}
			.parrafo_texto {
				font-family:camberlg;
				font-size:12pt;
				color:#000;
			}
</style>
