<template>
    <div id="app">
        <CoinCap />
    </div>
</template>
<script>
import CoinCap from './components/CoinCap'

export default {
    name: 'App',
    components: {
        CoinCap
    }
}
</script>
<style>
html,
body {
    margin: 0;
    font-family: 'Open Sans', sans-serif;
}
</style>