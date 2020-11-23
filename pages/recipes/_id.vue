<template>
  <section class="hero">
    <div class="hero-body">
      <div class="container">
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
                <div v-html="recipe.description" />
              </div>
            </div>
          </div>
        </section>
      </div>
    </div>
  </section>
</template>

<script>
import { request, gql, imageFields, seoMetaTagsFields } from '~/lib/datocms'
import { toHead } from 'vue-datocms'
import format from 'date-fns/format'
import parseISO from 'date-fns/parseISO'

export default {
  async asyncData({ params }) {
    const data = await request({
      query: gql`
        query RecipeQuery($slug: String!) {
          site: _site {
            favicon: faviconMetaTags {
              ...seoMetaTagsFields
            }
          }

          recipe(filter: { slug: { eq: $slug } }) {
            seo: _seoMetaTags {
              ...seoMetaTagsFields
            }
            id
            title
            publicationDate: _firstPublishedAt
            description
            image {
              responsiveImage(imgixParams: { fit: crop, ar: "16:9", w: 860 }) {
                ...imageFields
              }
            }
          }
        }

        ${imageFields}
        ${seoMetaTagsFields}
      `,
      variables: {
        slug: params.id
      }
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

    return toHead(this.recipe.seo, this.site.favicon)
  }
}
</script>
