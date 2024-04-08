<script lang="ts" setup>
import {Dices, HeartPulse, Menu, Table2} from 'lucide-vue-next'
import {Avatar, AvatarFallback, AvatarImage} from '@/components/ui/avatar'
import {Button} from '@/components/ui/button'
import {Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle} from '@/components/ui/card'
import {Sheet, SheetContent, SheetTrigger} from '@/components/ui/sheet'
import {Table, TableBody, TableCell, TableHead, TableHeader, TableRow} from '@/components/ui/table'
import {Popover, PopoverContent, PopoverTrigger,} from '@/components/ui/popover'
import {
  Pagination,
  PaginationEllipsis,
  PaginationFirst,
  PaginationLast,
  PaginationList,
  PaginationListItem,
  PaginationNext,
  PaginationPrev,
} from '@/components/ui/pagination'
import ThemeToggler from "@/components/ThemeToggler.vue";
import {ref} from "vue";
import dayjs from "dayjs";

const status = ref("Offline")
const totalAverage = ref(null)
const userAverages = ref(null)
const details = ref([])
const pageCount = ref(0)

async function fetchApiStatus() {
  try {
    const response = await fetch('https://rng-api.netrve.net/api/v1/Ping');
    const content = await response.json();
    if (content === "Pong") {
      status.value = "Online";
    }
  } catch (error) {
    console.error(error);
  }
}

async function fetchTotalAverage() {
  try {
    const response = await fetch('https://rng-api.netrve.net/api/v1/GetAverageRNG', {
      headers: {
        'Authorization': '3624af75-5ce0-4e2a-aee1-ab7086e7f317',
      }
    });
    const content = await response.json();
    if (content) {
      totalAverage.value = content;
      pageCount.value = Math.floor(content.count / 15)
    }
  } catch (error) {
    console.error(error);
  }
}

async function fetchUsers() {
  try {
    const response = await fetch('https://rng-api.netrve.net/api/v1/GetAllAveragesRNG', {
      headers: {
        'Authorization': '3624af75-5ce0-4e2a-aee1-ab7086e7f317',
      }
    });
    const content = await response.json();
    if (content) {
      userAverages.value = content.averages;
    }
  } catch (error) {
    console.error(error);
  }
}

async function fetchDetails(page: number) {
  try {
    const response = await fetch('https://rng-api.netrve.net/api/v1/GetGenerationDetails?' + new URLSearchParams({
      page: page.toString(),
      count: "15",
    }), {
      headers: {
        'Authorization': '3624af75-5ce0-4e2a-aee1-ab7086e7f317',
      }
    });

    const content = await response.json();
    if (content?.rngs) {
      details.value = content?.rngs.sort((timestamp: string) => dayjs(timestamp));
    }
  } catch (error) {
    console.error(error);
  }
}

fetchApiStatus()

if (status.value) {
  fetchTotalAverage()
  fetchUsers()
  fetchDetails(1)
}
</script>

<template>
  <div class="flex min-h-screen w-full flex-col">
    <header class="sticky top-0 flex h-16 items-center gap-4 border-b bg-background px-4 md:px-6">
      <nav
          class="hidden flex-col gap-6 text-lg font-medium md:flex md:flex-row md:items-center md:gap-5 md:text-sm lg:gap-6">
        <a
            class="flex items-center gap-2 text-lg font-semibold md:text-base"
            href="#"
        >
          <Table2 class="h-6 w-6"/>
          <span class="sr-only">RNG Service Overview</span>
        </a>
        <a
            class="text-foreground transition-colors hover:text-foreground"
            href="#"
        >
          Dashboard
        </a>
      </nav>
      <Sheet>
        <SheetTrigger as-child>
          <Button
              class="shrink-0 md:hidden"
              size="icon"
              variant="outline"
          >
            <Menu class="h-5 w-5"/>
            <span class="sr-only">Toggle navigation menu</span>
          </Button>
        </SheetTrigger>
        <SheetContent side="left">
          <nav class="grid gap-6 text-lg font-medium">
            <a
                class="flex items-center gap-2 text-lg font-semibold"
                href="#"
            >
              <Table2 class="h-6 w-6"/>
              <span class="sr-only">RNG Service Overview</span>
            </a>
            <a class="hover:text-foreground" href="#">
              Dashboard
            </a>
          </nav>
        </SheetContent>
      </Sheet>
      <div class="flex w-full items-center gap-4 md:ml-auto md:gap-2 lg:gap-4">
        <form class="ml-auto flex-1 sm:flex-initial">
        </form>
        <ThemeToggler/>
      </div>
    </header>
    <main class="flex flex-1 flex-col gap-4 p-4 md:gap-8 md:p-8" v-if="status == 'Offline'">
      <Card>
        <CardHeader class="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle class="text-sm font-medium">
            RNG API Status
          </CardTitle>
          <HeartPulse class="h-4 w-4 text-muted-foreground"/>
        </CardHeader>
        <CardContent>
          <div class="text-2xl font-bold">
            {{ status }}
          </div>
          <p class="text-xs text-muted-foreground">
            Current Status of the RNG API
          </p>
        </CardContent>
      </Card>
    </main>
    <main class="flex flex-1 flex-col gap-4 p-4 md:gap-8 md:p-8" v-else>
      <div class="grid gap-4 md:grid-cols-2 md:gap-8 lg:grid-cols-2">
        <Card>
          <CardHeader class="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle class="text-sm font-medium">
              RNG API Status
            </CardTitle>
            <HeartPulse class="h-4 w-4 text-muted-foreground"/>
          </CardHeader>
          <CardContent>
            <div class="text-2xl font-bold">
              {{ status }}
            </div>
            <p class="text-xs text-muted-foreground">
              Current Status of the RNG API
            </p>
          </CardContent>
        </Card>
        <Card>
          <CardHeader class="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle class="text-sm font-medium">
              Total Averages
            </CardTitle>
            <Dices class="h-4 w-4 text-muted-foreground"/>
          </CardHeader>
          <CardContent>
            <div class="text-2xl font-bold">
              {{ totalAverage?.average.toFixed(12) }}
            </div>
            <p class="text-xs text-muted-foreground">
              The average of all {{ totalAverage?.count }} generated numbers
            </p>
          </CardContent>
        </Card>
      </div>
      <div class="grid gap-4 md:gap-8 lg:grid-cols-2 xl:grid-cols-3">
        <Card class="xl:col-span-2">
          <CardHeader class="flex flex-row items-center justify-center">
            <div class="grid gap-2">
              <CardTitle>Generation Details</CardTitle>
              <CardDescription>
                Detailed insight about the generated numbers so far
              </CardDescription>
            </div>
          </CardHeader>
          <CardContent>
            <Table>
              <TableHeader>
                <TableRow>
                  <TableHead class="text-center">User</TableHead>
                  <TableHead class="text-center">Number</TableHead>
                  <TableHead class="text-center">Timestamp</TableHead>
                </TableRow>
              </TableHeader>
              <TableBody>
                <template v-for="detail in details" v-model="details">
                  <TableRow>
                    <TableCell>
                      <div class="font-medium">
                        {{ detail?.user }}
                      </div>
                    </TableCell>
                    <TableCell>
                      <Popover>
                        <PopoverTrigger class="p-1">
                          {{ detail?.rng.toFixed(53) }}
                        </PopoverTrigger>
                        <PopoverContent>
                          <div class="grid gap-2">
                            <CardTitle class="text-center">
                              <div class="text-xl font-bold">
                                Result Expressed as Roll
                              </div>
                              <p class="text-xs text-muted-foreground">
                                Formula: Math.ceil(number * dice)
                              </p></CardTitle>
                            <CardDescription>
                              <Table>
                                <TableHeader>
                                  <TableRow>
                                    <TableHead class="text-center">Dice</TableHead>
                                    <TableHead class="text-center">Result</TableHead>
                                  </TableRow>
                                </TableHeader>
                                <TableBody>
                                  <TableRow>
                                    <TableCell class="text-center">
                                      D4
                                    </TableCell>
                                    <TableCell class="text-center">
                                      {{ Math.ceil(detail?.rng * 4) }}
                                    </TableCell>
                                  </TableRow>
                                  <TableRow>
                                    <TableCell class="text-center">
                                      D6
                                    </TableCell>
                                    <TableCell class="text-center">
                                      {{ Math.ceil(detail?.rng * 6) }}
                                    </TableCell>
                                  </TableRow>
                                  <TableRow>
                                    <TableCell class="text-center">
                                      D8
                                    </TableCell>
                                    <TableCell class="text-center">
                                      {{ Math.ceil(detail?.rng * 8) }}
                                    </TableCell>
                                  </TableRow>
                                  <TableRow>
                                    <TableCell class="text-center">
                                      D10
                                    </TableCell>
                                    <TableCell class="text-center">
                                      {{ Math.ceil(detail?.rng * 10) }}
                                    </TableCell>
                                  </TableRow>
                                  <TableRow>
                                    <TableCell class="text-center">
                                      D12
                                    </TableCell>
                                    <TableCell class="text-center">
                                      {{ Math.ceil(detail?.rng * 12) }}
                                    </TableCell>
                                  </TableRow>
                                  <TableRow>
                                    <TableCell class="text-center">
                                      D20
                                    </TableCell>
                                    <TableCell class="text-center">
                                      {{ Math.ceil(detail?.rng * 20) }}
                                    </TableCell>
                                  </TableRow>
                                  <TableRow>
                                    <TableCell class="text-center">
                                      D100
                                    </TableCell>
                                    <TableCell class="text-center">
                                      {{ Math.ceil(detail?.rng * 100) }}
                                    </TableCell>
                                  </TableRow>
                                </TableBody>
                              </Table>
                            </CardDescription>
                          </div>
                        </PopoverContent>
                      </Popover>
                    </TableCell>
                    <TableCell>
                      {{ dayjs(detail?.timestamp).format('DD/MM/YYYY HH:mm:ss Z') }}
                    </TableCell>
                  </TableRow>
                </template>
              </TableBody>
            </Table>
            <CardFooter class="flex flex-row items-center justify-center">
              <Pagination v-slot="{ page }" :default-page="1" :itemsPerPage="15" :sibling-count="1"
                          :total="totalAverage?.count" show-edges @update:page="fetchDetails">
                <PaginationList v-slot="{ items }" class="flex items-center gap-1">
                  <PaginationFirst/>
                  <PaginationPrev/>
                  <template v-for="(item, index) in items">
                    <PaginationListItem v-if="item.type === 'page'" :key="index" :value="item.value" as-child>
                      <Button :variant="item.value === page ? 'default' : 'outline'" class="w-10 h-10 p-0">
                        {{ item.value }}
                      </Button>
                    </PaginationListItem>
                    <PaginationEllipsis v-else :key="item.type" :index="index"/>
                  </template>
                  <PaginationNext/>
                  <PaginationLast/>
                </PaginationList>
              </Pagination>
            </CardFooter>
          </CardContent>
        </Card>
        <Card>
          <CardHeader>
            <CardTitle>User Average Rolls</CardTitle>
          </CardHeader>
          <CardContent class="grid gap-8">
            <template v-for="userAverage in userAverages">
              <div class="flex items-center gap-4">
                <Avatar class="hidden h-9 w-9 sm:flex">
                  <AvatarImage alt="Avatar" src="/avatars/01.png"/>
                  <AvatarFallback>{{ userAverage?.user.substring(0, 2) + userAverage?.user.slice(-1) }}</AvatarFallback>
                </Avatar>
                <div class="grid gap-1">
                  <p class="text-sm font-medium leading-none">
                    {{ userAverage?.user }}
                  </p>
                  <p class="text-sm text-muted-foreground">
                    Roll Count: {{ userAverage?.count }}
                  </p>
                </div>
                <div class="ml-auto font-medium">
                  {{ userAverage?.average.toFixed(12) }}
                </div>
              </div>
            </template>
          </CardContent>
        </Card>
      </div>
    </main>
  </div>
</template>
