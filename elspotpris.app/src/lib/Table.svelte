<script>
    import { prices	} from "../routes/stores.js";

    export let height = 500;

    let priceData;

    const dayDateStringOptions = { weekday: 'long', day: "numeric" };
    const hourDateStringOptions = { hour: "2-digit" };

    function removeTime(date) {
        return new Date(date.toDateString());
    }

    const isActive = (date) => {
        return date.getHours() == new Date().getHours() && date.getDay() == new Date().getDay();
    }

    prices.subscribe((value) => {
        if (value)
        {
            const maxPrice = Math.max(...value.map(p => p[1]));
            const minPrice = Math.min(...value.map(p => p[1]));

            priceData = [];
            let day = null;
            value.forEach(p => {
                const severity = (p[1] - minPrice) / (maxPrice - minPrice);
                const red = ((severity * 200) | 0).toString(16).padStart(2,'0');
                const green = (((1 - severity) * 150) + 50 | 0).toString(16).padStart(2,'0');
                const blue = (((1 - Math.abs(severity - 0.5)) * 50) + 50 | 0).toString(16).padStart(2,'0');

                if (day == null || day.date.getTime() != removeTime(p[0]).getTime())
                {
                    day = { date: removeTime(p[0]), data: [] }
                    priceData.push(day);
                }

                day.data.push({
                    time: p[0], 
                    price: p[1],
                    color: `#${red}${green}${blue}`
                });
            });

            setActive();
        }
	});

    function setActive()
    {
        if (priceData)
        {
            priceData.forEach(d => {
                d.data.forEach(p => {
                    p.active = isActive(p.time);
                })
            });
        }
    }

    setInterval(() => {
        setActive();
	}, 10000);

</script>

<style lang="scss">
.price-table {
    display: flex;
    overflow: auto;
    flex-direction: column;

    .day {
        .hours {
            display: grid;
            grid-template-rows: repeat(12, auto);
            grid-gap: 1px .25em;
            grid-auto-flow: column;

            @media only screen and (min-width: 800px) {
                grid-template-rows: repeat(8, auto);
            }

            @media only screen and (min-width: 1200px) {
                grid-template-rows: repeat(6, auto);
            }

            @media only screen and (min-width: 1600px) {
                grid-gap: .25em 1px;
                grid-auto-flow: row;
                grid-template-columns: repeat(12, auto);
            }

            .hour {
                color: black;
                padding: 0.25em;
                font-size: 1.2em;
                &.active 
                {
                    border: 0.15em dashed blue;
                    padding: 0.25em 0.10em;
                    font-weight: bold;
                }
            }
        }
    }
}
</style>

{#if priceData}
<div class="price-table">
    {#each priceData as day}
    <div class="day">
        <p class="lead">{day.date.toLocaleDateString("da-DK", dayDateStringOptions)}</p>
        <div class="hours">
            {#each day.data as price}
                <div class="hour" class:active="{price.active}" style="background-color: {price.color}"><strong>{price.time.toLocaleTimeString("da-DK", hourDateStringOptions)}</strong> - {price.price} kr</div>
            {/each}
        </div>
    </div>
    {/each}
</div>
{/if}