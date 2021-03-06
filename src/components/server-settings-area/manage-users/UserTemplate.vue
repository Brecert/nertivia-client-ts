<template>
  <div
    class="server-member"
    @mouseover="hover = true"
    @mouseleave="hover = false"
    @contextmenu.prevent="rightClickEvent"
  >
    <AvatarImage
      class="avatar"
      :imageId="member.avatar"
      :seedId="member.id"
      size="30px"
      :animateGif="hover"
    />
    <span class="username" :style="{ color: firstRoleColor }">{{
      member.username
    }}</span>
    <div class="material-icons arrow">keyboard_arrow_right</div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import AvatarImage from "@/components/AvatarImage.vue";
import { ServerRolesModule } from "@/store/modules/serverRoles";
import ServerMember from "@/interfaces/ServerMember";
import User from "@/interfaces/User";
import ServerRole from "@/interfaces/ServerRole";
import { PopoutsModule } from "@/store/modules/popouts";

@Component({ components: { AvatarImage } })
export default class RightDrawer extends Vue {
  @Prop() private serverMember!: ServerMember & {
    member: User;
    roles: ServerRole[];
  };
  hover = false;

  rightClickEvent(event: MouseEvent) {
    PopoutsModule.ShowPopout({
      id: "context",
      component: "UserContextMenu",
      key: this.member.id + event.clientX + event.clientY,
      data: {
        x: event.clientX,
        y: event.clientY,
        id: this.member.id
      }
    });
  }

  get member() {
    return this.serverMember.member;
  }
  get firstRoleColor() {
    if (this.serverMember.roles[0]) {
      return this.serverMember.roles[0].color;
    }
    if (this.defaultRole && this.defaultRole.color) {
      return this.defaultRole.color;
    }
    return undefined;
  }
  get defaultRole() {
    return ServerRolesModule.defaultServerRole(this.serverMember.server_id);
  }
}
</script>

<style lang="scss" scoped>
.server-member {
  display: flex;
  align-content: center;
  align-items: center;
  border-radius: 4px;
  height: 40px;
  flex-shrink: 0;
  margin-left: 5px;
  cursor: pointer;
  user-select: none;
  margin-right: 5px;
  transition: 0.2s;
  &:hover {
    background: rgba(255, 255, 255, 0.1);
    .arrow {
      opacity: 1;
    }
  }
}
.avatar {
  margin-right: 5px;
  margin-left: 5px;
}
.arrow {
  margin-left: auto;
  transition: 0.2s;
  opacity: 0.1;
}
</style>
