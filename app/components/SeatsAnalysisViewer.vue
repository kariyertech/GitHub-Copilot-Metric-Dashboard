<template>
    <div class="tiles-container">
        <v-card
elevation="4" color="white" variant="elevated" class="mx-auto my-4"
            style="width: 330px; height: 175px; cursor: pointer;"
            @click="showUsersModal('total')">
            <v-card-item class="d-flex justify-center align-center">
                <div class="tiles-text">
                    <div class="text-overline mb-1" style="visibility: hidden;">filler</div>
                    <div class="text-h6 mb-1">Toplam Atanan Lisans</div>
                    <div class="text-caption">
                        Şu anda atanmış lisanslar
                    </div>
                    <p class="text-h4">{{ totalSeats.length }}</p>
                </div>
            </v-card-item>
        </v-card>

        <v-card
elevation="4" color="white" variant="elevated" class="mx-auto my-3"
            style="width: 300px; height: 175px; cursor: pointer;"
            @click="showUsersModal('noshow')">
            <v-card-item class="d-flex justify-center align-center">
                <div class="tiles-text">
                    <div class="text-overline mb-1" style="visibility: hidden;">filler</div>
                    <div class="text-h6 mb-1">Atandı Ama Hiç Kullanılmadı</div>
                    <div class="text-caption">
                        Hiç kullanılmayan lisanslar
                    </div>
                    <p class="text-h4">{{ noshowSeats }}</p>
                </div>
            </v-card-item>
        </v-card>
        <v-card elevation="4" color="white" variant="elevated" class="mx-auto my-4" style="width: 330px; height: 175px; cursor: pointer;"
            @click="showUsersModal('7days')">
            <v-card-item class="d-flex justify-center align-center">
                <div class="tiles-text">
                    <div class="text-overline mb-1" style="visibility: hidden;">filler</div>
                    <div class="text-h6 mb-1">Son 7 Günde Aktivite Yok</div>
                    <div class="text-caption">
                        Son 7 günde kullanılmadı
                    </div>
                    <p class="text-h4">{{ unusedSeatsInSevenDays }}</p>
                </div>
            </v-card-item>
        </v-card>
        <v-card elevation="4" color="white" variant="elevated" class="mx-auto my-4" style="width: 330px; height: 175px; cursor: pointer;"
            @click="showUsersModal('30days')">
            <v-card-item class="d-flex justify-center align-center">
                <div class="tiles-text">
                    <div class="text-overline mb-1" style="visibility: hidden;">filler</div>
                    <div class="text-h6 mb-1">Son 30 Günde Aktivite Yok</div>
                    <div class="text-caption">
                        Son 30 günde kullanılmadı
                    </div>
                    <p class="text-h4">{{ unusedSeatsInThirtyDays }}</p>
                </div>
            </v-card-item>
        </v-card>
    </div>
    
    <!-- User List Modal -->
    <v-dialog v-model="showModal" max-width="500px">
      <v-card>
        <v-card-title class="text-h5">
          {{ modalTitle }}
        </v-card-title>
        <v-card-text>
          <v-list>
            <v-list-item v-for="user in modalUsers" :key="user.id">
              <v-list-item-content>
                <v-list-item-title>{{ user.login }}</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list>
          <div v-if="modalUsers.length === 0" class="text-center py-4">
            Hiç kullanıcı bulunamadı
          </div>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="showModal = false">
            Kapat
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <div>
        <v-main class="p-1" style="min-height: 300px;">
            <v-container style="min-height: 300px;" class="px-4 elevation-2">
                <br>
                <h2>Tüm Atanmış Lisanslar</h2>
                <br>
            <v-data-table :headers="headers" :items="totalSeats" :items-per-page="10" class="elevation-2">
                <template #item="{ item, index }">
                    <tr>
                        <td>{{ index + 1 }}</td>
                        <td>{{ item.login }}</td>
                        <td>{{ item.id }}</td>
                        <td>{{ item.team }}</td>
                        <td>{{ item.created_at }}</td>
                        <td>{{ item.last_activity_at }}</td>
                        <td>{{ item.last_activity_editor }}</td>
                    </tr>
                </template>
                </v-data-table>
            </v-container>
        </v-main>
    </div>
</template>
  
<script lang="ts">
  import { defineComponent, ref, watchEffect } from 'vue';
  import type { Seat } from '@/model/Seat';
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
name: 'SeatsAnalysisViewer',
props: {
        seats: {
            type: Array as () => Seat[],
            required: true,
            default: () => []  
        }
    },
setup(props) {
    const totalSeats = ref<Seat[]>([]);
        const noshowSeats = ref<number>(0);
        const unusedSeatsInSevenDays = ref<number>(0);
        const unusedSeatsInThirtyDays = ref<number>(0);
        const showModal = ref(false);
        const modalTitle = ref('');
        const modalUsers = ref<Seat[]>([]);

        let noshowCount = 0;
        let unusedIn7Count = 0;
        let unusedIn30Count = 0;

        const showUsersModal = (type: string) => {
          let users: Seat[] = [];
          let title = '';
          
          const oneWeekAgo = new Date();
          const thirtyDaysAgo = new Date();
          oneWeekAgo.setDate(oneWeekAgo.getDate() - 7);
          thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
          
          switch(type) {
            case 'total':
              users = totalSeats.value;
              title = 'Tüm Atanmış Lisanslar';
              break;
            case 'noshow':
              users = totalSeats.value.filter(seat => !Boolean(seat.last_activity_at));
              title = 'Hiç Kullanılmayan Lisanslar';
              break;
            case '7days':
              users = totalSeats.value.filter(seat => {
                if (!seat.last_activity_at) return false;
                const lastActivityDate = new Date(seat.last_activity_at);
                return lastActivityDate < oneWeekAgo;
              });
              title = 'Son 7 Günde Aktivite Olmayan Kullanıcılar';
              break;
            case '30days':
              users = totalSeats.value.filter(seat => {
                if (!seat.last_activity_at) return false;
                const lastActivityDate = new Date(seat.last_activity_at);
                return lastActivityDate < thirtyDaysAgo;
              });
              title = 'Son 30 Günde Aktivite Olmayan Kullanıcılar';
              break;
          }
          
          modalUsers.value = users;
          modalTitle.value = title;
          showModal.value = true;
        };

        watchEffect(() => {
            if (props.seats && Array.isArray(props.seats)) {
                totalSeats.value = props.seats;

                const oneWeekAgo = new Date();
                const thirtyDaysAgo = new Date();
                oneWeekAgo.setDate(oneWeekAgo.getDate() - 7);
                thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

                noshowCount = 0;
                unusedIn7Count = 0;
                unusedIn30Count = 0;

                props.seats.forEach(seat => {
                    if(!Boolean(seat.last_activity_at)) {
                        noshowCount++;
                    } else {
                        const lastActivityDate = new Date(seat.last_activity_at);
                        if (lastActivityDate < oneWeekAgo) {
                            unusedIn7Count++;
                        }
                        if (lastActivityDate < thirtyDaysAgo) {
                            unusedIn30Count++;
                        }
                    }
                });

                // to sort totalSeats by last_activity_at
                totalSeats.value.sort((a, b) => {
                    if (a.last_activity_at === null) {
                        return -1;
                    }
                    if (b.last_activity_at === null) {
                        return 1;
                    }
                    return new Date(a.last_activity_at) > new Date(b.last_activity_at) ? 1 : -1;
                });

                noshowSeats.value = noshowCount;
                unusedSeatsInSevenDays.value = unusedIn7Count;
                unusedSeatsInThirtyDays.value = unusedIn30Count;
            } else {
                throw new Error('Invalid number of seats');
            }
        });

        return {
            totalSeats,
            noshowSeats: noshowSeats,
            unusedSeatsInSevenDays: unusedSeatsInSevenDays,
            unusedSeatsInThirtyDays: unusedSeatsInThirtyDays,
            showModal,
            modalTitle,
            modalUsers,
            showUsersModal
        }
},
data() {
    return {
        headers: [
            { title: 'S.No', key: 'serialNumber'},
            { title: 'Kullanıcı Adı', key: 'login' },
            { title: 'GitHub ID', key: 'id' },
            { title: 'Atayan Takım', key: 'team' },
            { title: 'Atanma Zamanı', key: 'created_at' },
            { title: 'Son Aktivite', key: 'last_activity_at' },
            { title: 'Son Kullanılan Editör', key: 'last_activity_editor' },
        ],
    };
}   
  
});
</script>
