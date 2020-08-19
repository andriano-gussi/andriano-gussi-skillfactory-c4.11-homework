<template>
	<b-alert
      variant="danger"
      dismissible
      fade
      :show="showDismissibleAlert"
      @dismissed="updating"
    >
      {{ message }}
    </b-alert>
</template>

<script>
export default {
  props: ['message', 'showDismissibleAlert'],
  methods: {
		updating() {
			this.$emit('showAlert')
		}
  }
};
</script>
