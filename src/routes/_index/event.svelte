<script>
  import { t } from 'svelte-i18n';
  import { createEventDispatcher, onMount, tick } from 'svelte';
  import { mdiChevronRight } from '@mdi/js';
  import dayjs from 'dayjs';
  import duration from 'dayjs/plugin/duration';
  dayjs.extend(duration);

  import Icon from '../../components/Icon.svelte';
  import { eventsData } from '../../data/timeline';

  const dispatch = createEventDispatcher();

  const now = dayjs();
  let upcoming = [];
  let current = [];

  function checkEvent(event) {
    if (!event.showOnHome) return;

    const start = dayjs(event.start);
    const end = dayjs(event.end);
    if (start.isBefore(now) && end.isAfter(now)) {
      const diff = end.diff(now);
      const timeLeft = dayjs.duration(diff);
      event.time = timeLeft.format(diff > 86400000 ? 'D[d] H[h]' : 'H[h]');
      current = [...current, event];
    } else if (start.isAfter(now)) {
      const diff = start.diff(now);
      const timeUpcoming = dayjs.duration(diff);
      event.time = timeUpcoming.format(diff > 86400000 ? 'D[d] H[h]' : 'H[h]');
      upcoming = [...upcoming, event];
    }
  }

  function parseEvents() {
    for (const event of eventsData) {
      if (Array.isArray(event)) {
        for (const ev of event) {
          checkEvent(ev);
        }
      } else {
        checkEvent(event);
      }
    }
  }

  onMount(async () => {
    parseEvents();
    await tick();
    dispatch('done');
  });
</script>

<div class="bg-item rounded-xl p-4 flex flex-col">
  {#if upcoming.length > 0}
    <p class="text-white mb-1">{$t('home.event.upcoming')}</p>
    <div class="flex flex-col">
      {#each upcoming as item}
        <div class="pl-2 pr-1 py-1 rounded-xl mb-1 flex" style="background: {item.color};">
          <span class="whitespace-no-wrap overflow-x-hidden flex-1 mr-1 text-sm" style="text-overflow: ellipsis;">
            {item.name}
          </span>
          <span class="bg-black bg-opacity-50 rounded-xl px-2 text-white text-sm">{item.time}</span>
        </div>
      {/each}
    </div>
  {/if}
  {#if current.length > 0}
    <p class="text-white mb-1">{$t('home.event.current')}</p>
    <div class="flex flex-col">
      {#each current as item}
        <div class="pl-2 pr-1 py-1 rounded-xl mb-1 flex" style="background: {item.color};">
          <span class="whitespace-no-wrap overflow-x-hidden flex-1 mr-1 text-sm" style="text-overflow: ellipsis;">
            {item.name}
          </span>
          <span class="bg-black bg-opacity-50 rounded-xl px-2 text-white text-sm">{item.time}</span>
        </div>
      {/each}
    </div>
  {/if}
  <a
    href="/timeline"
    class="flex justify-end items-center self-end lg:self-start text-white mt-4 
    bg-background-secondary rounded-xl py-2 px-4 hover:bg-background transition-colors duration-100"
  >
    <img src="/images/timeline.png" alt="wish" class="mr-2 h-6 w-6" />
    {$t('home.event.detail')}
    <Icon path={mdiChevronRight} />
  </a>
</div>
