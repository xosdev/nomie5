<script>
  import { Lang } from '../../store/lang'
  import { UserStore } from '../../store/user-store'
  import dayjs from 'dayjs'
  import Button from '../../components/button/button.svelte'

</script>

<section class="slide slide-time-format">
  <div class="pt-3 mx-auto top center-grow" style="width:300px;">
    <h1 class="mb-4 text-2xl font-bold text-center dark:text-white">
      {Lang.t('setup.choose-time-format', `Which time looks right?`)}
    </h1>

    <Button
      className="mb-3 {!$UserStore.meta.is24Hour ? 'bg-primary-500 text-white font-bold' : 'bg-white dark:bg-black text-primary-600'}"
      block
      size="lg"
      type="clear"
      delay={20}
      on:click={() => {
        $UserStore.meta.is24Hour = false
        UserStore.saveMeta()
      }}>
      {dayjs().format('h:mm a')}
    </Button>
    <Button
      className="mb-3 {$UserStore.meta.is24Hour ? 'bg-primary-500 text-white font-bold' : 'bg-white dark:bg-black text-primary-600'}"
      type="clear"
      block
      size="lg"
      delay={20}
      on:click={() => {
        $UserStore.meta.is24Hour = true
        UserStore.saveMeta()
      }}>
      {dayjs().format('HH:mm')}
    </Button>
  </div>

</section>
