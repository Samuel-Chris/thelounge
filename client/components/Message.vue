<template>
	<div
		:id="'msg-' + message.id"
		:class="['msg', message.type, {self: message.self, highlight: message.highlight}]"
		:data-from="message.from && message.from.nick"
	>
		<span :aria-label="message.time | localetime" class="time tooltipped tooltipped-e"
			>{{ messageTime }}
		</span>
		<template v-if="message.type === 'unhandled'">
			<span class="from">[{{ message.command }}]</span>
			<span class="content">
				<span v-for="(param, id) in message.params" :key="id">{{ param }} </span>
			</span>
		</template>
		<template v-else-if="isAction()">
			<span class="from"><span class="only-copy">*** </span></span>
			<Component :is="messageComponent" :network="network" :message="message" />
		</template>
		<template v-else-if="message.type === 'action'">
			<span class="from"><span class="only-copy">* </span></span>
			<span class="content" dir="auto">
				<Username :user="message.from" dir="auto" />&#32;<ParsedMessage
					:network="network"
					:message="message"
				/>
				<LinkPreview
					v-for="preview in message.previews"
					:key="preview.link"
					:keep-scroll-position="keepScrollPosition"
					:link="preview"
				/>
			</span>
		</template>
		<template v-else>
			<span v-if="message.type === 'message'" class="from">
				<template v-if="message.from && message.from.nick">
					<span class="only-copy">&lt;</span>
					<Username :user="message.from" />
					<span class="only-copy">&gt; </span>
				</template>
			</span>
			<span v-else-if="message.type === 'plugin'" class="from">
				<template v-if="message.from && message.from.nick">
					<span class="only-copy">[</span>
					{{ message.from.nick }}
					<span class="only-copy">] </span>
				</template>
			</span>
			<span v-else class="from">
				<template v-if="message.from && message.from.nick">
					<span class="only-copy">-</span>
					<Username :user="message.from" />
					<span class="only-copy">- </span>
				</template>
			</span>
			<span class="content" dir="auto">
				<ParsedMessage :network="network" :message="message" />
				<LinkPreview
					v-for="preview in message.previews"
					:key="preview.link"
					:keep-scroll-position="keepScrollPosition"
					:link="preview"
				/>
			</span>
		</template>
	</div>
</template>

<script>
import Username from "./Username.vue";
import LinkPreview from "./LinkPreview.vue";
import ParsedMessage from "./ParsedMessage.vue";
import MessageTypes from "./MessageTypes";

const moment = require("moment");
const constants = require("../js/constants");

MessageTypes.ParsedMessage = ParsedMessage;
MessageTypes.LinkPreview = LinkPreview;
MessageTypes.Username = Username;

export default {
	name: "Message",
	components: MessageTypes,
	props: {
		message: Object,
		network: Object,
		keepScrollPosition: Function,
	},
	computed: {
		messageTime() {
			const format = this.$root.settings.showSeconds
				? constants.timeFormats.msgWithSeconds
				: constants.timeFormats.msgDefault;

			return moment(this.message.time).format(format);
		},
		messageComponent() {
			return "message-" + this.message.type;
		},
	},
	mounted() {
		require("../js/renderPreview");
	},
	methods: {
		isAction() {
			return typeof MessageTypes["message-" + this.message.type] !== "undefined";
		},
	},
};
</script>
