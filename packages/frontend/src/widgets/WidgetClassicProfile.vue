<template>
<div class="_panel">
	<MkContainer>
			<div :class="$style.banner" :style="{ backgroundImage: $i.bannerUrl ? `url(${ $i.bannerUrl })` : null}" @click="changeBanner"></div>
			<div :class="$style.avatarContainer">
				<MkAvatar :class="$style.avatar" :user="$i" @click="changeAvatar"/>
				<MkA :class="$style.name" :to="userPage($i)" >
						<MkUserName :user="$i" style="font-weight: bold;"/>
					</MkA>
					
					<MkAcct :user="$i" :class="$style.name"/>
			</div>

		</MkContainer>
</div>
</template>

<script lang="ts" setup>
import { useWidgetPropsManager, Widget, WidgetComponentExpose } from './widget';
import { GetFormResultType } from '@/scripts/form';
import { $i } from '@/account';
import { userPage } from '@/filters/user';
import { i18n } from '@/i18n';
import { selectFile } from '@/scripts/select-file';
import * as os from '@/os';

const name = 'classicProfile';

const widgetPropsDef = {
};

type WidgetProps = GetFormResultType<typeof widgetPropsDef>;

// 現時点ではvueの制限によりimportしたtypeをジェネリックに渡せない
//const props = defineProps<WidgetComponentProps<WidgetProps>>();
//const emit = defineEmits<WidgetComponentEmits<WidgetProps>>();
const props = defineProps<{ widget?: Widget<WidgetProps>; }>();
const emit = defineEmits<{ (ev: 'updateProps', props: WidgetProps); }>();

const { widgetProps, configure } = useWidgetPropsManager(name,
	widgetPropsDef,
	props,
	emit,
);

defineExpose<WidgetComponentExpose>({
	name,
	configure,
	id: props.widget ? props.widget.id : null,
});

function changeAvatar(ev) {
	selectFile(ev.currentTarget ?? ev.target, i18n.ts.avatar).then(async (file) => {
		let originalOrCropped = file;

		const { canceled } = await os.confirm({
			type: 'question',
			text: i18n.t('cropImageAsk'),
		});

		if (!canceled) {
			originalOrCropped = await os.cropImage(file, {
				aspectRatio: 1,
			});
		}

		const i = await os.apiWithDialog('i/update', {
			avatarId: originalOrCropped.id,
		});
		$i.avatarId = i.avatarId;
		$i.avatarUrl = i.avatarUrl;
	});
}

function changeBanner(ev) {
	selectFile(ev.currentTarget ?? ev.target, i18n.ts.banner).then(async (file) => {
		let originalOrCropped = file;

		const { canceled } = await os.confirm({
			type: 'question',
			text: i18n.t('cropImageAsk'),
		});

		if (!canceled) {
			originalOrCropped = await os.cropImage(file, {
				aspectRatio: 2,
			});
		}

		const i = await os.apiWithDialog('i/update', {
			bannerId: originalOrCropped.id,
		});
		$i.bannerId = i.bannerId;
		$i.bannerUrl = i.bannerUrl;
	});
}
</script>

<style lang="scss" module>
.container {
	position: relative;
	background-size: cover;
	background-position: center;
	display: flex;
}

.banner {
	height: 100px;
	background-size: cover;
	background-position: center;
	cursor: pointer;
	background-color: #4c5e6d;
	box-shadow: 0 0 128px rgb(0 0 0 / 50%) inset;
}

.avatarContainer {
	padding-bottom: 16px;
}

.avatar {
	display: block;
		position: absolute;
		top: 76px;
		left: 16px;
		width: 58px;
		height: 58px;
		border: solid 3px var(--face);
		border-radius: 8px;
		cursor: pointer;

}

.bodyContainer {
	display: flex;
	align-items: center;
	min-width: 0;
	padding: 0 16px 0 0;
}

.body {
	text-overflow: ellipsis;
	overflow: clip;
}

.name {
	display: block;
		margin: 10px 0 0 84px;
		line-height: 16px;
		color: var(--fg);
		overflow: hidden;
		text-overflow: ellipsis;
}

.username {
	display: block;
		margin: 4px 0 8px 84px;
		line-height: 16px;
		font-size: 0.9em;
		color: var(--fg);
		opacity: 0.7;

}
</style>
