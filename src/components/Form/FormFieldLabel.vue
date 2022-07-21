<template>
	<label :for="controlId" class="">
		<div><br></div>
		<div class="título_nivel_2">
			<template v-if="localeLabel">
				<span class="título_nivel_2">{{ localeLabel }}</span>
				<span class="título_nivel_2">{{ multilingualLabel }}</span>
			</template>
			<template v-else>
				<div  class="titulo_nivel_2">{{ label }}</div>
			</template>
		</div>
		<span v-if="isRequired" class="pkpFormFieldLabel__required">		
			<span class="aclaraciones_co">* Campo obligatorio</span>
		</span>
		<div><br></div>
	</label>
	
</template>

<script>
export default {
	name: 'FormFieldLabel',
	props: {
		controlId: String,
		label: String,
		localeLabel: String,
		multilingualLabel: String,
		isRequired: {
			type: Boolean,
			default: false
		},
		requiredLabel: String
	}
};
</script>

<style lang="less">
@import '../../styles/_import';

.pkpFormFieldLabel {
	font-weight: @bold;
}
.aclaraciones_co {
				font-family:camberuli;
				font-size:8;
				color:#ED0000;
}
.título_nivel_2 {
	font-family:cambersb;
	font-size:18pt;
	color:#737373;
}
			
.pkpFormFieldLabel__required {
	color: @no;
}
</style>
