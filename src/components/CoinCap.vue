<template class="coins">
    <table class="coins__table"
           v-if="assets && assets.length">
        <thead class="coins__head">
            <tr>
                <th class="coins__cell coins__cell--text-left">Название</th>
                <th class="coins__cell">Стоимость</th>
                <th class="coins__cell coins__cell--d-xs-none">Капитализация</th>
                <th class="coins__cell coins__cell--d-xs-none">Объем</th>
            </tr>
        </thead>
        <div class="coins__body-wrapper">
            <tbody class="coins__body">
                <tr v-for="(it, i) in assets"
                    :key="i">
                    <td class="coins__cell coins__cell--text-left">{{ it.name }}</td>
                    <td class="coins__cell">{{ (+it.priceUsd).toFixed(2) }}</td>
                    <td class="coins__cell coins__cell--d-xs-none">{{ (+it.marketCapUsd).toFixed(2) }}</td>
                    <td class="coins__cell coins__cell--d-xs-none">{{ (+it.volumeUsd24Hr).toFixed(2) }}</td>
                </tr>
            </tbody>
        </div>
    </table>
</template>
<script>
export default {
    data() {
        return {
            assets: [],
        };
    },
    computed: {
        coinList() {
            return this.assets.map(it => it.name.toLowerCase().replace(/ /g, '-')).join(',')
        }
    },
    async created() {
        await this.getAssets()
        this.getPrices()
    },
    methods: {
        getAssets() {
            return new Promise((resolve, reject) => {
                this.$http.get('https://api.coincap.io/v2/assets').then(res => {
                    let data = res.data.data
                        .sort((a, b) => b.marketCapUsd - a.marketCapUsd)
                        .slice(0, 15)
                    this.assets = data
                    resolve()
                })
            })
        },
        getPrices() {
            const pricesWs = new WebSocket(`wss://ws.coincap.io/prices?assets=${this.coinList}`)
            pricesWs.onmessage = (prices) => {
                let data = JSON.parse(prices.data)
                for (let key in data) {
                    this.assets.forEach((it) => {
                        if (key.replace(/-/g, ' ').toLowerCase() === it.name.toLowerCase()) {
                            it.priceUsd = data[key]
                            it.marketCapUsd = it.priceUsd * it.supply
                        }
                    })

                }
            }
        }
    }
};
</script>
<style lang="scss"
       scoped>
.coins {
    &__table {
        width: 100%;
        text-align: right;
        border-collapse: collapse;
    }

    &__head,
    &__body tr {
        display: table;
        width: 100%;
        table-layout: fixed;
    }

    &__head {
        color: rgba(0, 0, 0, 0.5);
        width: calc(100% - 17px);
        position: relative;

        &::after {
            content: '';
            display: block;
            width: calc(100% + 17px);
            position: absolute;
            bottom: 0;
            left: 0;
            border-bottom: 1px solid #eaeaea;
        }
    }

    &__body-wrapper {
        height: calc(400px);
        overflow: auto;
        visibility: hidden;

        &:hover {
            visibility: visible;
        }
    }

    &__body {
        display: block;
        visibility: visible;
    }

    &__cell {
        padding: 15px;

        &--text-left {
            text-align: left;
        }

        &--d-xs-none {
            @media (max-width: 700px) {
                display: none;
            }
        }
    }
}
</style>