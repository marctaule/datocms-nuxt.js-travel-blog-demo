<template>
  <div>
    <section class="hero">
      <div class="hero-body">
        <div class="container">
          <div v-for="recipe in recipes.slice(0, 2)" v-bind:key="recipe.slug">
            <section class="section">
              <div class="columns">
                <div class="column is-8 is-offset-2">
                  <div class="content is-medium">
                    <h2 class="subtitle is-4">
                      {{ formatDate(recipe.publicationDate) }}
                    </h2>
                    <h1 class="title">
                      <nuxt-link :to="`/recipes/${recipe.slug}`">{{
                        recipe.title
                      }}</nuxt-link>
                    </h1>
                  </div>
                </div>
              </div>
            </section>

            <div class="is-divider" />
          </div>
        </div>
      </div>
    </section>

  </div>
</template>

<script>
import { request, gql, seoMetaTagsFields } from '~/lib/datocms'
import { toHead } from 'vue-datocms'
import format from 'date-fns/format'
import parseISO from 'date-fns/parseISO'

export default {
  async asyncData({ params }) {
    const data = await request({
      query: gql`
        {
          site: _site {
            favicon: faviconMetaTags {
              ...seoMetaTagsFields
            }
          }

          recipes: allRecipes(first: 10, orderBy: _firstPublishedAt_DESC) {
            id
            title
            slug
            publicationDate: _firstPublishedAt
          }
        }
        ${seoMetaTagsFields}
      `
    })

    return { ready: !!data, ...data }
  },
  methods: {
    formatDate(date) {
      return format(parseISO(date), 'PPP')
    }
  },
  head() {
    if (!this.ready) {
      return
    }

    return toHead(this.site.favicon)
  }
}
</script>
