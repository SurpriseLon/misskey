<template>
<MkSpacer :content-max="1000">
	<div ref="rootEl" class="edbbcaef">
		<MkFolder class="item">
			<template #header>Stats</template>
			<XStats/>
		</MkFolder>
		<MkFolder class="item">
			<template #header>Active users</template>
			<XActiveUsers/>
		</MkFolder>
		<MkFolder class="item">
			<template #header>Moderators</template>
			<XModerators/>
		</MkFolder>
		<MkFolder class="item">
			<template #header>Federation</template>
			<XFederation/>
		</MkFolder>
		<MkFolder class="item">
			<template #header>Instances</template>
			<XInstances/>
		</MkFolder>
		<MkFolder class="item">
			<template #header>Ap requests</template>
			<XApRequests/>
		</MkFolder>
		<MkFolder class="item">
			<template #header>New users</template>
			<XUsers/>
		</MkFolder>
		<MkFolder class="item">
			<template #header>Deliver queue</template>
			<XQueue domain="deliver"/>
		</MkFolder>
		<MkFolder class="item">
			<template #header>Inbox queue</template>
			<XQueue domain="inbox"/>
		</MkFolder>
	</div>
</MkSpacer>
</template>

<script lang="ts" setup>
import { markRaw, version as vueVersion, onMounted, onBeforeUnmount, nextTick } from 'vue';
import XFederation from './overview.federation.vue';
import XInstances from './overview.instances.vue';
import XQueue from './overview.queue.vue';
import XApRequests from './overview.ap-requests.vue';
import XUsers from './overview.users.vue';
import XActiveUsers from './overview.active-users.vue';
import XStats from './overview.stats.vue';
import XModerators from './overview.moderators.vue';
import MkTagCloud from '@/components/MkTagCloud.vue';
import { version, url } from '@/config';
import * as os from '@/os';
import { stream } from '@/stream';
import { i18n } from '@/i18n';
import { definePageMetadata } from '@/scripts/page-metadata';
import 'chartjs-adapter-date-fns';
import { defaultStore } from '@/store';
import MkFileListForAdmin from '@/components/MkFileListForAdmin.vue';
import MkFolder from '@/components/MkFolder.vue';

const rootEl = $ref<HTMLElement>();
let serverInfo: any = $ref(null);
let topSubInstancesForPie: any = $ref(null);
let topPubInstancesForPie: any = $ref(null);
let federationPubActive = $ref<number | null>(null);
let federationPubActiveDiff = $ref<number | null>(null);
let federationSubActive = $ref<number | null>(null);
let federationSubActiveDiff = $ref<number | null>(null);
let newUsers = $ref(null);
let activeInstances = $shallowRef(null);
const queueStatsConnection = markRaw(stream.useChannel('queueStats'));
const now = new Date();
const filesPagination = {
	endpoint: 'admin/drive/files' as const,
	limit: 9,
	noPaging: true,
};

function onInstanceClick(i) {
	os.pageWindow(`/instance-info/${i.host}`);
}

onMounted(async () => {
	/*
	const magicGrid = new MagicGrid({
		container: rootEl,
		static: true,
		animate: true,
	});

	magicGrid.listen();
	*/

	os.apiGet('charts/federation', { limit: 2, span: 'day' }).then(chart => {
		federationPubActive = chart.pubActive[0];
		federationPubActiveDiff = chart.pubActive[0] - chart.pubActive[1];
		federationSubActive = chart.subActive[0];
		federationSubActiveDiff = chart.subActive[0] - chart.subActive[1];
	});

	os.apiGet('federation/stats', { limit: 10 }).then(res => {
		topSubInstancesForPie = res.topSubInstances.map(x => ({
			name: x.host,
			color: x.themeColor,
			value: x.followersCount,
			onClick: () => {
				os.pageWindow(`/instance-info/${x.host}`);
			},
		})).concat([{ name: '(other)', color: '#80808080', value: res.otherFollowersCount }]);
		topPubInstancesForPie = res.topPubInstances.map(x => ({
			name: x.host,
			color: x.themeColor,
			value: x.followingCount,
			onClick: () => {
				os.pageWindow(`/instance-info/${x.host}`);
			},
		})).concat([{ name: '(other)', color: '#80808080', value: res.otherFollowingCount }]);
	});

	os.api('admin/server-info').then(serverInfoResponse => {
		serverInfo = serverInfoResponse;
	});

	os.api('admin/show-users', {
		limit: 5,
		sort: '+createdAt',
	}).then(res => {
		newUsers = res;
	});

	os.api('federation/instances', {
		sort: '+lastCommunicatedAt',
		limit: 25,
	}).then(res => {
		activeInstances = res;
	});

	nextTick(() => {
		queueStatsConnection.send('requestLog', {
			id: Math.random().toString().substr(2, 8),
			length: 100,
		});
	});
});

onBeforeUnmount(() => {
	queueStatsConnection.dispose();
});

const headerActions = $computed(() => []);

const headerTabs = $computed(() => []);

definePageMetadata({
	title: i18n.ts.dashboard,
	icon: 'ti ti-dashboard',
});
</script>

<style lang="scss" scoped>
.edbbcaef {
	display: grid;
	grid-template-columns: repeat(auto-fill, minmax(400px, 1fr));
	grid-gap: 16px;
}
</style>
