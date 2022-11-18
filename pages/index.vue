<template>
  <div>
    <div v-if="resultVisible" class="mr-result-container">
      <div class="nhsuk-width-container">
        <div class="nhsuk-grid-row">
          <div class="nhsuk-grid-column-two-thirds">
            <div class="preferred-style-result">
              <h1 class="nhsuk-heading-xl">Your preferred style:</h1>
              <div class="preferred-style-container">
                <p>{{ preferredStyle }}</p>
              </div>
            </div>
          </div>
          <div class="nhsuk-grid-column-one-third">
            <dl class="nhsuk-summary-list nhsuk-summary-list--no-border">
              <div
                v-for="(result, option) in results"
                :key="'result-' + option"
                class="nhsuk-summary-list__row"
              >
                <dt class="nhsuk-summary-list__key nhsuk-u-font-size-24">
                  {{ option }}
                </dt>
                <dd class="nhsuk-summary-list__value">
                  <div
                    class="result-bar"
                    :class="preferredStyle == option ? 'preferred' : ''"
                    :style="
                      'width:' + (result.barLength / maxBarLength) * 100 + '%;'
                    "
                  >
                    &nbsp;
                  </div>
                </dd>
              </div>
            </dl>
            <button class="nhsuk-button" @click="clearSelection">
              Restart
            </button>
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      <h1>
        Merrill & Reid Personal Style Inventory
        <span class="nhsuk-caption-m">
          Source: David Merrill & Roger Reid,
          <em>Personal Styles and Effective Performance</em>, 1981
        </span>
      </h1>
      <p>
        <span class="nhsuk-tag nhsuk-tag--yellow">
          Privacy notice: none of the information you provide on this page is
          collected or shared in any form
        </span>
      </p>
      <p>Select the word or phrase in each set of 4 that is most like you:</p>
      <ul class="nhsuk-grid-row nhsuk-card-group">
        <li
          v-for="(set, index) in phraseSets.body"
          :key="index"
          class="nhsuk-grid-column-one-third nhsuk-card-group__item"
        >
          <div
            class="nhsuk-card"
            :class="selectedPhrases[index] != null ? 'completed' : ''"
          >
            <div class="nhsuk-card__content">
              <div class="nhsuk-form-group">
                <fieldset class="nhsuk-fieldset">
                  <div class="nhsuk-radios">
                    <div
                      v-for="(phrase, option) in set"
                      :key="set + '-' + option"
                      class="nhsuk-radios__item"
                    >
                      <input
                        :id="'phrase-set-' + index + '-' + option"
                        v-model="selectedPhrases[index]"
                        class="nhsuk-radios__input"
                        :name="'phrase-set-' + index + '-' + option"
                        type="radio"
                        :value="[option, phrase]"
                      />
                      <label
                        class="nhsuk-label nhsuk-radios__label"
                        :for="'phrase-set-' + index + '-' + option"
                      >
                        {{ phrase }}
                      </label>
                    </div>
                  </div>
                </fieldset>
              </div>
            </div>
          </div>
        </li>
      </ul>
      <div class="nhsuk-grid-row">
        <div class="nhsuk-grid-column-one-half">
          <div class="nhsuk-card">
            <div class="nhsuk-card__content">
              <p class="nhsuk-heading-l">
                {{ selectedCount }}/{{ phraseCount }} selected
              </p>
              <p v-if="!allSelected">
                Make sure you've selected a phrase from every set then return
                here to see your results
              </p>
              <button
                v-else
                class="nhsuk-button nhsuk-u-font-size-48 nhsuk-u-padding-6"
                :disabled="!allSelected"
                @click="viewResult"
              >
                See results
              </button>
            </div>
          </div>
        </div>
        <div class="nhsuk-grid-column-one-half">
          <button
            class="nhsuk-button nhsuk-button--secondary"
            @click="clearSelection"
          >
            Clear selection
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  async asyncData({ $content, error }) {
    const phraseSets = await $content('phrase-sets')
      .only(['body'])
      .fetch()
      .catch((err) => {
        error({ statusCode: 404, message: err })
      })

    const phraseCount = phraseSets.body.length

    return {
      phraseSets,
      phraseCount,
    }
  },
  data() {
    return {
      selectedPhrases: [],
      resultVisible: false,
    }
  },
  head: {
    title: 'Merrill & Reid Personal Style Inventory',
    script: [
      {
        src: 'https://identity.netlify.com/v1/netlify-identity-widget.js',
      },
    ],
  },
  computed: {
    selectedCount() {
      return this.selectedPhrases.filter((p) => p !== null).length
    },
    allSelected() {
      const completed = this.selectedCount === this.phraseCount
      return completed
    },
    results() {
      const options = ['A', 'B', 'C', 'D']
      const weighting = {
        A: [
          11, 16, 23, 32, 38, 46, 54, 62, 67, 72, 77, 85, 92, 96, 99, 103, 106,
          110, 113,
        ],
        B: [
          4, 17, 21, 35, 43, 51, 62, 71, 76, 81, 86, 94, 101, 105, 108, 111,
          115,
        ],
        C: [
          15, 22, 38, 45, 49, 64, 68, 73, 81, 86, 92, 96, 99, 103, 107, 110,
          114,
        ],
        D: [11, 24, 32, 41, 47, 67, 70, 79, 86, 93, 99, 103, 106, 110, 114],
      }
      const selectedCounts = {
        A: { count: 0, barLength: 0 },
        B: { count: 0, barLength: 0 },
        C: { count: 0, barLength: 0 },
        D: { count: 0, barLength: 0 },
      }
      for (let i = 0; i < options.length; i++) {
        const count = this.selectedPhrases.filter(
          (p) => p[0] === options[i]
        ).length
        const maxPossible = weighting[options[i]][weighting[options[i]].length-1]
        const barLength = weighting[options[i]][count] ?? maxPossible
        selectedCounts[options[i]].count = count
        selectedCounts[options[i]].barLength = barLength
      }
      return selectedCounts
    },
    preferredStyle() {
      const sorted = Object.entries(this.results).sort(
        (a, b) => b[1].barLength - a[1].barLength
      )
      return sorted[0][0]
    },
    maxBarLength() {
      const sorted = Object.entries(this.results).sort(
        (a, b) => b[1].barLength - a[1].barLength
      )
      return sorted[0][1].barLength
    },
  },
  methods: {
    clearSelection() {
      this.selectedPhrases = []
      this.resultVisible = false
    },
    viewResult() {
      this.resultVisible = true
    },
  },
}
</script>

<style lang="scss">
@import 'node_modules/nhsuk-frontend/packages/components/skip-link/skip-link';
@import 'node_modules/nhsuk-frontend/packages/components/card/card';
@import 'node_modules/nhsuk-frontend/packages/components/tag/tag';
@import 'node_modules/nhsuk-frontend/packages/components/fieldset/fieldset';
@import 'node_modules/nhsuk-frontend/packages/components/radios/radios';
@import 'node_modules/nhsuk-frontend/packages/components/button/button';
@import 'node_modules/nhsuk-frontend/packages/components/summary-list/summary-list';

.nhsuk-card.completed {
  border-color: $color_nhsuk-green;
  background-color: tint($color_nhsuk-green, 80%);
}

.nhsuk-radios__item {
  font-weight: $nhsuk-font-bold;
}

.result-bar {
  background-color: $color_nhsuk-grey-3;

  &.preferred {
    background-color: $color_nhsuk-blue;
  }
}

.preferred-style-result {
  text-align: center;

  .preferred-style-container {
    padding: 16px;
    background-color: $color_nhsuk-pink;
    margin-bottom: 32px;

    p {
      font-size: 200px;
      vertical-align: top;
      font-weight: $nhsuk-font-bold;
      color: $color_nhsuk-white;
    }
  }
}
</style>
