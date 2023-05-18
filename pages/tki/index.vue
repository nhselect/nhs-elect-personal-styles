<template>
  <div>
    <div v-if="resultVisible" class="mr-result-container">
      <div class="nhsuk-width-container">
        <div class="nhsuk-grid-row">
          <div class="nhsuk-grid-column-one-half">
            <div class="preferred-style-result">
              <h1 class="nhsuk-heading-xl">
                Your preferred style:
              </h1>
              <div class="preferred-style-container">
                <p class="nhsuk-u-font-size-64">
                  {{ statements[preferredStyle] }}
                </p>
              </div>
            </div>
          </div>
          <div class="nhsuk-grid-column-one-half">
            <dl class="nhsuk-summary-list nhsuk-summary-list--no-border">
              <div
                v-for="(result, option) in results"
                :key="'result-' + option"
                class="nhsuk-summary-list__row"
              >
                <dt class="nhsuk-summary-list__key nhsuk-u-font-size-20">
                  {{ statements[option] }} {{ result.count }}
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
        Thomas Kilman Conflict Mode Instrument (TKI)
        <span class="nhsuk-caption-m">
          Source: Kenneth L. Thomas and Ralph H. Kilman
        </span>
      </h1>
      <p>
        <span class="nhsuk-tag nhsuk-tag--yellow">
          Privacy notice: none of the information you provide on this page is
          collected or shared in any form
        </span>
      </p>
      <p>Consider situations in which you find your wishes differing from those of another person. How do you usually respond to such situations?</p>
      <p>Below are several pairs of statements describing possible behavioural responses. For each pair, select the statement which is most characteristic of your own behaviour.</p>
      <p>In many cases, neither statement may be very typical of your behaviour, but please select the response which you would be more likely to use.</p>
      <ul class="nhsuk-grid-row nhsuk-card-group">
        <li
          v-for="(set, index) in statementSets.sets"
          :key="set.number"
          class="nhsuk-grid-column-three-quarters nhsuk-card-group__item"
        >
          <div
            class="nhsuk-card"
            :class="selectedStatements[index] != null ? 'completed' : ''"
          >
            <div class="nhsuk-card__content">
              <div class="nhsuk-form-group">
                <fieldset class="nhsuk-fieldset">
                  <div class="nhsuk-radios">
                    <div
                      v-for="(statement, option) in set.options"
                      :key="set + '-' + option"
                      class="nhsuk-radios__item"
                    >
                      <input
                        :id="'statement-set-' + index + '-' + option"
                        v-model="selectedStatements[index]"
                        class="nhsuk-radios__input"
                        :name="'statement-set-' + index + '-' + option"
                        type="radio"
                        :value="[index, statement.text]"
                      />
                      <label
                        class="nhsuk-label nhsuk-radios__label"
                        :for="'statement-set-' + index + '-' + option"
                      >
                        {{ statement.text }}
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
                {{ selectedCount }}/{{ statementCount }} selected
              </p>
              <p v-if="!allSelected">
                Make sure you've selected a statement from every set then return
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
    const statementSets = await $content('tki/statement-sets')
      .fetch()
      .catch((err) => {
        error({ statusCode: 404, message: err })
      })

    const statementCount = statementSets.sets.length

    return {
      statementSets,
      statementCount,
    }
  },
  data() {
    return {
      selectedStatements: [],
      resultVisible: false,
      statements: {
        'competing': 'Competing (forcing)',
        'collaborating': 'Collaborating (problem solving)',
        'compromising': 'Compromising (sharing)',
        'avoiding': 'Avoiding (withdrawal)',
        'accommodating': 'Accommodating (soothing)'
      }
    }
  },
  head: {
    title: 'Thomas Kilman Conflict Mode Instrument (TKI)',
  },
  computed: {
    selectedCount() {
      return this.selectedStatements.filter((p) => p !== null).length
    },
    allSelected() {
      const completed = this.selectedCount === this.statementCount
      return completed
    },
    results() {
      const options = [
        'competing', 'collaborating', 'compromising', 'avoiding', 'accommodating'
      ]
      const selectedCounts = {
        competing: { count: 0, barLength: 0 },
        collaborating: { count: 0, barLength: 0 },
        compromising: { count: 0, barLength: 0 },
        avoiding: { count: 0, barLength: 0 },
        accommodating: { count: 0, barLength: 0 },
      }
      for (let i = 0; i < this.selectedStatements.length; i++) {
        const qn = this.statementSets.sets[this.selectedStatements[i][0]]
        const ans = this.selectedStatements[i][1]
        const char = qn.options.find(o => o.text === ans).behaviour

        selectedCounts[char].count += 1
        selectedCounts[char].barLength += 1
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
      this.selectedStatements = []
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
      vertical-align: top;
      font-weight: $nhsuk-font-bold;
      color: $color_nhsuk-white;
    }
  }
}
</style>