<template>
	<input
		type="search"
		class="form-control"
		@keyup="searchPhraseKeyUp"
		:id="inputId"
		:value="searchPhrase"
		:placeholder="currentSearchLabel"
	/>
</template>

<script>
import debounce from 'debounce';

export default {
	props: {
		searchLabel: {
			type: String,
			default() {
				return '';
			}
		},
		searchPhrase: {
			type: String,
			default() {
				return '';
			}
		}
	},
	computed: {
		inputId() {
			return this._uid;
		},
		currentSearchLabel() {
			return this.searchLabel ? this.searchLabel : this.__('common.search');
		}
	},
	methods: {
		/**
		 * Emit an event when the search phrase changes in response to a keyup
		 * event in the input field.
		 *
		 * @param {Object} data A DOM event (object) or the new search
		 *  phrase (string)
		 */
		searchPhraseKeyUp: debounce(function(el) {
			this.$emit('search-phrase-changed', el.target.value);
		}, 250),

		/**
		 * Clear the search phrase
		 */
		clearSearchPhrase() {
			this.$emit('search-phrase-changed', '');
			this.$nextTick(function() {
				this.$el.querySelector('input[type="search"]').focus();
			});
		}
	}
};
</script>

<style lang="less">
@import '../../styles/_import';

.pkpSearch {
	position: relative;
	width: 18rem;
	max-width: 100%;
}

.pkpSearch__input,
/* Override legacy form styles for: .pkp_form input[type="search"] */
.pkp_form .pkpSearch .pkpSearch__input {
	display: block;
	box-sizing: border-box;
	padding: 0 0.5em 0 3.5em;
	width: 100%;
	height: auto;
	border: @bg-border-light;
	border-radius: @radius;
	font-size: @font-sml;
	line-height: 2rem;
	-webkit-appearance: none; /* Override Safari search input styles */

	&:hover {
		border-color: @primary;

		+ .pkpSearch__icons {
			border-color: @primary;
		}
	}

	&:focus {
		outline: 0;
		border-color: @primary;

		+ .pkpSearch__icons {
			border-color: @primary;
			background: @primary;

			.pkpSearch__icons--search:before {
				color: #fff;
			}
		}
	}
}

.pkpSearch__clear {
	position: absolute;
	top: 0;
	right: 0;
	width: 2rem;
	height: 100%;
	background: transparent;
	border: none;
	border-top-right-radius: @radius;
	border-bottom-right-radius: @radius;
	vertical-align: middle;
	text-align: center;
	color: @offset;
	cursor: pointer;

	&:hover,
	&:focus {
		outline: 0;
		background: @offset;
		color: #fff;
	}
}

.pkpSearch__icons {
	position: absolute;
	top: 0;
	left: 0;
	width: 2.5em;
	height: 100%;
	border-right: @bg-border-light;
	border-top-left-radius: @radius;
	border-bottom-left-radius: @radius;
}

.pkpSearch__icons--search {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	color: @primary;
}
</style>
