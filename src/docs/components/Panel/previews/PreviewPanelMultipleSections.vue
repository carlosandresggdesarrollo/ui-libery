<template>
	<panel>
		<panel-section>
			<template slot="header">
				<h2>Journal Overview</h2>
				<p>
					The following details about your journal are provided for your review.
					Please review them carefully before proceeding.
				</p>
			</template>
			<list>
				<list-item>
					<template slot="value">
						<icon icon="comment-o" :inline="true" />
						32
					</template>
					Number of active discussions in this journal.
				</list-item>
				<list-item>
					<template slot="value">
						<icon icon="clock-o" :inline="true" />
						67
					</template>
					Days until next issue is published.
				</list-item>
				<list-item>
					<icon icon="exclamation-triangle" :inline="true" />
					This journal is not currently available for the public to view. Would
					you like to
					<a href="#">make it public</a>
					?
				</list-item>
			</list>
		</panel-section>
		<panel-section>
			<template slot="header">
				<h2>Journal Overview</h2>
				<p>
					The following details about your journal are provided for your review.
					Please review them carefully before proceeding.
				</p>
			</template>
			<list>
				<list-item>
					<template slot="value">
						<icon icon="comment-o" :inline="true" />
						32
					</template>
					Number of active discussions in this journal.
				</list-item>
				<list-item>
					<template slot="value">
						<icon icon="clock-o" :inline="true" />
						67
					</template>
					Days until next issue is published.
				</list-item>
				<list-item>
					<icon icon="exclamation-triangle" :inline="true" />
					This journal is not currently available for the public to view. Would
					you like to
					<a href="#">make it public</a>
					?
				</list-item>
			</list>
		</panel-section>
	</panel>
</template>

<script>
import List from '@/components/List/List.vue';
import ListItem from '@/components/List/ListItem.vue';

export default {
	components: {
		List,
		ListItem
	}
};
</script>
