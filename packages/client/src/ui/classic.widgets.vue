<template>
<div class="ddiqwdnk">
	<XWidgets class="widgets" :edit="editMode" :widgets="$store.reactiveState.widgets.value.filter(w => w.place === place)" @add-widget="addWidget" @remove-widget="removeWidget" @update-widget="updateWidget" @update-widgets="updateWidgets" @exit="editMode = false"/>
	<MkAd class="a" :prefer="['square']"/>

	<button v-if="editMode" class="_textButton edit" style="font-size: 0.9em;" @click="editMode = false"><i class="ti ti-check"></i> {{ $ts.editWidgetsExit }}</button>
	<button v-else class="_textButton edit" style="font-size: 0.9em;" @click="editMode = true"><i class="ti ti-pencil"></i> {{ $ts.editWidgets }}</button>
</div>
</template>

<script lang="ts">
import { defineComponent, defineAsyncComponent } from 'vue';
import XWidgets from '@/components/MkWidgets.vue';

export default defineComponent({
	components: {
		XWidgets,
	},

	props: {
		place: {
			type: String,
		},
	},

	emits: ['mounted'],

	data() {
		return {
			editMode: false,
		};
	},

	mounted() {
		this.$emit('mounted', this.$el);
	},

	methods: {
		addWidget(widget) {
			this.$store.set('widgets', [{
				...widget,
				place: this.place,
			}, ...this.$store.state.widgets]);
		},

		removeWidget(widget) {
			this.$store.set('widgets', this.$store.state.widgets.filter(w => w.id !== widget.id));
		},

		updateWidget({ id, data }) {
			this.$store.set('widgets', this.$store.state.widgets.map(w => w.id === id ? {
				...w,
				data,
			} : w));
		},

		updateWidgets(widgets) {
			this.$store.set('widgets', [
				...this.$store.state.widgets.filter(w => w.place !== this.place),
				...widgets,
			]);
		},
	},
});
</script>

<style lang="scss" scoped>
.ddiqwdnk {
	position: sticky;
	height: min-content;
	box-sizing: border-box;
	padding-bottom: 8px;

	> .widgets,
	> .a {
		width: 300px;
	}

	> .edit {
		display: block;
		margin: 16px auto;
	}
}
</style>
