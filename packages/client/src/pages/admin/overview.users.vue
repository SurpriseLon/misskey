<template>
<div :class="$style.root">
	<transition :name="$store.state.animation ? 'zoom' : ''" mode="out-in">
		<MkLoading v-if="fetching"/>
		<div v-else class="users">
			<MkA v-for="(user, i) in newUsers" :key="user.id" :to="`/user-info/${user.id}`" class="user">
				<MkUserCardMini :user="user"/>
			</MkA>
		</div>
	</transition>
</div>
</template>

<script lang="ts" setup>
import { onMounted, onUnmounted, ref } from 'vue';
import * as os from '@/os';
import { useInterval } from '@/scripts/use-interval';
import MkUserCardMini from '@/components/MkUserCardMini.vue';

let newUsers = $ref(null);
let fetching = $ref(true);

const fetch = async () => {
	const _newUsers = await os.api('admin/show-users', {
		limit: 5,
		sort: '+createdAt',
		origin: 'local',
	});
	newUsers = _newUsers;
	fetching = false;
};

useInterval(fetch, 1000 * 60, {
	immediate: true,
	afterMounted: true,
});
</script>

<style lang="scss" module>
.root {
	&:global {
		> .users {
			.chart-move {
				transition: transform 1s ease;
			}

			display: grid;
			grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
			grid-gap: 12px;

			> .user:hover {
				text-decoration: none;
			}
		}
	}
}
</style>
