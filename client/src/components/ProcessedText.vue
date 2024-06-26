<template>
	<span data-cy="processed-text">
		<span v-for="(item, index) in content" :key="index">
			<span v-if="item.type === 'text'">{{ item.text }}</span>
			<a
				v-else-if="item.type === 'link'"
				class="link text-primary"
				:href="item.text"
				@click="e => onLinkClick(e, item.text)"
			>
				<span>
					{{ item.text }}
					<v-tooltip top activator="parent" v-if="showAddQueueTooltip">
						<span>{{ $t("processed-text.link-hint") }}</span>
					</v-tooltip>
				</span>
			</a>
		</span>
	</span>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch, Ref, toRefs } from "vue";

const urlRegex = /(https?:\/\/[^\s]+)/;

interface ContentItem {
	type: "text" | "link";
	text: string;
}

const props = withDefaults(
	defineProps<{
		text: string;
		showAddQueueTooltip: boolean;
	}>(),
	{
		showAddQueueTooltip: true,
	}
);

const emit = defineEmits<{
	"link-click": [link: string];
}>();

const { text } = toRefs(props);
const content: Ref<ContentItem[]> = ref([]);

function onLinkClick(e: Event, link: string) {
	if (props.showAddQueueTooltip) {
		e.preventDefault();
		e.stopPropagation();
		emit("link-click", link);
	}
}

function processText() {
	content.value = [];
	if (!text.value) {
		return;
	}
	let match;
	let index = 0;
	let loop = 0;
	while ((match = urlRegex.exec(text.value.substring(index))) !== null) {
		// console.log("msg:", this.text, "match", match, "content", this.content);
		if (match.index > index) {
			content.value.push({
				type: "text",
				text: text.value.slice(index, index + match.index),
			});
		}
		content.value.push({ type: "link", text: match[0] });
		index += match.index + match[0].length;
		loop++;
		if (loop > 10) {
			break;
		}
	}
	if (index < text.value.length) {
		content.value.push({ type: "text", text: text.value.substring(index) });
	}
}

onMounted(() => {
	processText();
});

watch(
	() => text,
	() => {
		processText();
	}
);
</script>

<style lang="scss" scoped>
@import "../variables.scss";
</style>
