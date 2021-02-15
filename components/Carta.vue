<template>
    <b-col md="3">
        <b-card
        :title="title"
        style="max-width:20rem"
        class="mb-2"
        tag="article"
        >
          <b-card-text>
            {{ text }}
          </b-card-text>
          <b-button variant="primary">Apply</b-button>
        </b-card>
      </b-col>
</template>

<script>
export default {
    name: "Carta",
    props: ['id', 'title', 'text']
}
</script>