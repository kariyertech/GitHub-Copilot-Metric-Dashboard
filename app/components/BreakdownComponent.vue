<template>
  <div>
    <div class="tiles-container">
      <!-- Acceptance Rate Tile -->  
      <v-card elevation="4" color="white" variant="elevated" class="mx-auto my-3" style="width: 300px; height: 175px;">
          <v-card-item>
            <div class="tiles-text">
              <div class="spacing-25"/>
              <div class="text-h6 mb-1">{{ breakdownCountTitle }}</div>
              <div class="text-caption">
                Son 28 günde
              </div>
              <p class="text-h4">{{ numberOfBreakdowns }}</p> 
          </div>
        </v-card-item>
      </v-card>
    </div>

    <v-main class="p-1" style="min-height: 300px;">
      <v-container style="min-height: 300px;" class="px-4 elevation-2">
        <v-row>
          <v-col cols="4">
            <v-card>
              <v-card-item class="d-flex justify-center align-center">
                <div class="spacing-25"/>
                <div class="text-h6 mb-1">En Çok Kabul Edilen 5 {{ breakdownDisplayNamePlural }}</div>
                <div style="width: 300px; height: 300px;">
                  <Pie :data="breakdownsChartDataTop5AcceptedPrompts" :options="chartOptions" />
                </div>
              </v-card-item>
            </v-card>
          </v-col>

          <v-col cols="4">
            <v-card>
              <v-card-item class="d-flex justify-center align-center">
                <div class="spacing-25"/>
                <div class="text-h6 mb-1">En İyi 5 {{ breakdownDisplayNamePlural }} Kabul Oranı (Adet)</div>
                <div style="width: 300px; height: 300px;">
                  <Pie :data="breakdownsChartDataTop5AcceptedPromptsByCounts" :options="chartOptions" />
                </div>
              </v-card-item>
            </v-card>
          </v-col>

          <v-col cols="4">
            <v-card>
              <v-card-item class="d-flex justify-center align-center">
                <div class="spacing-25"/>
                <div class="text-h6 mb-1">En İyi 5 {{ breakdownDisplayNamePlural }} Kabul Oranı (Satır)</div>
                <div style="width: 300px; height: 300px;">
                  <Pie :data="breakdownsChartDataTop5AcceptedPromptsByLines" :options="chartOptions" />
                </div>
              </v-card-item>
            </v-card>
          </v-col>
        </v-row>

        <br>
        <h2>{{ breakdownDisplayNamePlural }} Dağılımı</h2>
        <br>

        <v-data-table :headers="headers" :items="breakdownList" class="elevation-2" style="padding-left: 100px; padding-right: 100px;">
            <template #item="{item}">
                <tr>
                    <td>{{ item.name }}</td>
                    <td>{{ item.acceptedPrompts }}</td>
                    <td>{{ item.suggestedPrompts }}</td>
                    <td>{{ item.acceptedLinesOfCode }}</td>
                    <td>{{ item.suggestedLinesOfCode }}</td>
                    <td v-if="item.acceptanceRateByCount !== undefined">{{ item.acceptanceRateByCount.toFixed(2) }}%</td>
                    <td v-if="item.acceptanceRateByLines !== undefined">{{ item.acceptanceRateByLines.toFixed(2) }}%</td>
                </tr>
            </template>
        </v-data-table>
      </v-container>
    </v-main>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, toRef } from 'vue';
import type { Metrics } from '@/model/Metrics';
import { Breakdown } from '@/model/Breakdown';
import { Pie } from 'vue-chartjs'

import {
  Chart as ChartJS,
  ArcElement,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  BarElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js'

ChartJS.register(
  ArcElement, 
  CategoryScale,
  LinearScale,
  BarElement,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
)

export default defineComponent({
  name: 'BreakdownComponent',
  components: {
    Pie
  },
  props: {
      metrics: {
          type: Object,
          required: true
      },
      breakdownKey: {
          type: String,
          required: true
      }
  },
  setup(props) {

    // Create a reactive reference to store the breakdowns.
    const breakdownList = ref<Breakdown[]>([]);

    // Number of breakdowns
    const numberOfBreakdowns = ref(0);

    // Breakdowns Chart Data for breakdowns breakdown Pie Chart
    const breakdownsChartData = ref<{ labels: string[]; datasets: any[] }>({ labels: [], datasets: [] });

    //Top 5 by accepted prompts
    const breakdownsChartDataTop5AcceptedPrompts = ref<{ labels: string[]; datasets: any[] }>({ labels: [], datasets: [] });

    //Acceptance Rate by lines for top 5 by accepted prompts
    const breakdownsChartDataTop5AcceptedPromptsByLines = ref<{ labels: string[]; datasets: any[] }>({ labels: [], datasets: [] });

    //Acceptance Rate by counts for top 5 by accepted prompts
    const breakdownsChartDataTop5AcceptedPromptsByCounts = ref<{ labels: string[]; datasets: any[] }>({ labels: [], datasets: [] });

    const chartOptions = {
      responsive: true,
      maintainAspectRatio: true,
    };

    const pieChartColors = ref([
    '#4B0082', // Indigo
    '#41B883', // Vue Green
    '#6495ED', // Cornflower Blue
    '#87CEFA', // Light Sky Blue
    '#7CFC00'  // Lawn Green
]);

    const data = toRef(props, 'metrics').value;

    // Process the breakdown separately
    data.forEach((m: Metrics) => m.breakdown.forEach(breakdownData => 
    {
      const breakdownName = breakdownData[props.breakdownKey as keyof typeof breakdownData] as string;
      let breakdown = breakdownList.value.find(b => b.name === breakdownName);

      if (!breakdown) {
        // Create a new breakdown object if it does not exist
        breakdown = new Breakdown({
          name: breakdownName,
          acceptedPrompts: breakdownData.acceptances_count,
          suggestedPrompts: breakdownData.suggestions_count,
          suggestedLinesOfCode: breakdownData.lines_suggested,
          acceptedLinesOfCode: breakdownData.lines_accepted,
        });
        breakdownList.value.push(breakdown);
      } else {
        // Update the existing breakdown object
        breakdown.acceptedPrompts += breakdownData.acceptances_count;
        breakdown.suggestedPrompts += breakdownData.suggestions_count;
        breakdown.suggestedLinesOfCode += breakdownData.lines_suggested;
        breakdown.acceptedLinesOfCode += breakdownData.lines_accepted;
      }
      // Recalculate the acceptance rates
      breakdown.acceptanceRateByCount = breakdown.suggestedPrompts !== 0 ? (breakdown.acceptedPrompts / breakdown.suggestedPrompts) * 100 : 0;
      breakdown.acceptanceRateByLines = breakdown.suggestedLinesOfCode !== 0 ? (breakdown.acceptedLinesOfCode / breakdown.suggestedLinesOfCode) * 100 : 0;

      // Log each breakdown for debugging
     // console.log('Breakdown:', breakdown);
    }));

    //Sort breakdowns map by accepted prompts
    breakdownList.value.sort((a, b) => b.acceptedPrompts - a.acceptedPrompts);

    // Get the top 5 breakdowns by accepted prompts
    const top5BreakdownsAcceptedPrompts = breakdownList.value.slice(0, 5);
    
    breakdownsChartDataTop5AcceptedPrompts.value = {
      labels: top5BreakdownsAcceptedPrompts.map(breakdown => breakdown.name),
      datasets: [
        {
          data: top5BreakdownsAcceptedPrompts.map(breakdown => breakdown.acceptedPrompts),
          backgroundColor: pieChartColors.value,
        },
      ],
    };

    breakdownsChartDataTop5AcceptedPromptsByLines.value = {
      labels: top5BreakdownsAcceptedPrompts.map(breakdown => breakdown.name),
      datasets: [
        {
          data: top5BreakdownsAcceptedPrompts.map(breakdown => breakdown.acceptanceRateByLines.toFixed(2)),
          backgroundColor: pieChartColors.value,
        },
      ],
    };

    breakdownsChartDataTop5AcceptedPromptsByCounts.value = {
      labels: top5BreakdownsAcceptedPrompts.map(breakdown => breakdown.name),
      datasets: [
        {
          data: top5BreakdownsAcceptedPrompts.map(breakdown => breakdown.acceptanceRateByCount.toFixed(2)),
          backgroundColor: pieChartColors.value,
        },
      ],
    };

    numberOfBreakdowns.value = breakdownList.value.length;

    return { chartOptions, breakdownList, numberOfBreakdowns, 
      breakdownsChartData, breakdownsChartDataTop5AcceptedPrompts, breakdownsChartDataTop5AcceptedPromptsByLines, breakdownsChartDataTop5AcceptedPromptsByCounts };
  },
  computed: {
    breakdownDisplayName() {
      return this.breakdownKey === 'language' ? 'Dil' : 
             this.breakdownKey === 'editor' ? 'Editör' : 
             this.breakdownKey.charAt(0).toUpperCase() + this.breakdownKey.slice(1);
    },
    breakdownDisplayNamePlural() {
      return this.breakdownKey === 'language' ? 'Diller' : 
             this.breakdownKey === 'editor' ? 'Editörler' : 
             `${this.breakdownDisplayName}s`;
    },
    breakdownCountTitle() {
      return this.breakdownKey === 'language' ? 'Kullanılan Dil Sayısı' : 
             this.breakdownKey === 'editor' ? 'Kullanılan Editör Sayısı' : 
             `${this.breakdownDisplayNamePlural} Sayısı`;
    },
    headers() {
      return [
        { title: `${this.breakdownDisplayName} Adı`, key: 'name' },
        { title: 'Kabul Edilen Öneriler', key: 'acceptedPrompts' },
        { title: 'Önerilen Prompts', key: 'suggestedPrompts' },
        { title: 'Kabul Edilen Kod Satırı', key: 'acceptedLinesOfCode' },
        { title: 'Önerilen Kod Satırı', key: 'suggestedLinesOfCode' },
        { title: 'Kabul Oranı (Adet) (%)', key: 'acceptanceRateByCount' },
        { title: 'Kabul Oranı (Satır) (%)', key: 'acceptanceRateByLines' },
      ];
    },
  },
  

});
</script>
