<template>
	<div>
		<div v-if="conversations">
			<div
				v-for="(conversation, index) in conversations"
				:key="conversation.name"
				ref="conversationContainer"
				class="flex flex-col"
			>
				<div :ref="`conversation-${index}`">
					<ConversationCard
						:user-name="getUserName(conversation)"
						:profile-pic-url="
							conversation.sender.image ? conversation.sender.image : ''
						"
						:time="conversation.creation"
						:message="conversation.content"
						:attachments="conversation.attachments"
						:color="getConversationCardColor(conversation)"
					/>
					<!-- TODO: dynamically set color based on user -->
				</div>
			</div>
		</div>
		<div v-else class="p-5">
			<LoadingText text="Fetching conversations..." />
		</div>
	</div>
</template>

<script>
import ConversationCard from "./ConversationCard.vue";
import { LoadingText } from "frappe-ui";
import { ref } from "vue";

export default {
	name: "Conversations",
	components: {
		ConversationCard,
		LoadingText,
	},
	props: ["ticketId", "scrollToBottom"],
	setup() {
		const userColors = ref({});
		const lastColorIndex = ref(-1);

		return { userColors, lastColorIndex };
	},
	resources: {
		conversations() {
			return {
				url: "helpdesk.api.ticket.get_conversations",
				params: {
					ticket_id: this.ticketId,
				},
				auto: true,
			};
		},
	},
	computed: {
		conversations() {
			this.$nextTick(() => {
				this.autoScrollToBottom();
			});
			return this.$resources.conversations.data || null;
		},
	},
	watch: {
		scrollToBottom(scroll) {
			if (scroll) {
				this.autoScrollToBottom();
			}
		},
	},
	mounted() {
		this.$socket.on("helpdesk:new-communication", (data) => {
			if (data.ticket_id !== this.ticketId) return;
			this.$resources.conversations.fetch();
		});
	},
	unmounted() {
		this.$socket.off("helpdesk:new-communication");
	},
	methods: {
		getUserName(conversation) {
			return (
				(conversation.sender.first_name ? conversation.sender.first_name : "") +
				" " +
				(conversation.sender.last_name ? conversation.sender.last_name : "")
			);
		},
		autoScrollToBottom() {
			if (this.conversations) {
				const [el] =
					this.$refs["conversation-" + (this.conversations.length - 1)];
				if (el) {
					el.scrollIntoView({ behavior: "smooth" });
				}
			}
		},
		getConversationCardColor(conversation) {
			const userName = this.getUserName(conversation);
			let cardColors = ["blue", "gray", "green", "red"];

			if (!this.userColors[userName]) {
				if (this.lastColorIndex == cardColors.length - 1) {
					this.lastColorIndex = 0;
				} else {
					this.lastColorIndex++;
				}
				this.userColors[userName] = cardColors[this.lastColorIndex];
			}
			return this.userColors[userName];
		},
	},
};
</script>

<style></style>
