<script lang="ts" setup>
import { onUnmounted } from "vue";
import { useConnection } from "@/plugins/connection";
import { useRoomApi } from "@/util/roomapi";
import { useStore } from "@/store";

const connection = useConnection();
const roomapi = useRoomApi(connection);

const store = useStore();
const loginUnsub = store.subscribe(mutation => {
	if (mutation.type === "LOGIN" || mutation.type === "LOGOUT") {
		roomapi.notify("usernameChanged");
	}
});

onUnmounted(() => {
	loginUnsub();
});
</script>
