<script setup lang="ts">
import { ref, reactive, computed, onMounted } from "vue";
import cardData from "../Hanafuda_Card_Data.json";

enum CardType {
    LIGHT = "LIGHT",
    SEED = "SEED",
    RIBBON = "RIBBON",
    CHAFF = "CHAFF",
}

interface Card {
    month: number;
    type: CardType;
    name: string;
    icon: string | null;
}

interface Yaku {
    [key: string]: number | { base: number; additional: number };
}

interface CalculatedScore {
    totalScore: number;
    yaku: Yaku;
}

const selectedCards = ref<Card[]>([]);
const chaffCount = ref(0);

const availableLights: Card[] = cardData.lights.map((card) => ({
    ...card,
    type: CardType.LIGHT,
}));

const availableSeeds: Card[] = cardData.seeds.map((card) => ({
    ...card,
    type: CardType.SEED,
}));

const availableRibbons: Card[] = cardData.ribbons.map((card) => ({
    ...card,
    type: CardType.RIBBON,
}));

const isSelected = (card: Card) =>
    selectedCards.value.some(
        (c) => c.month === card.month && c.type === card.type
    );

const toggleCard = (card: Card) => {
    if (isSelected(card)) {
        selectedCards.value = selectedCards.value.filter(
            (c) => !(c.month === card.month && c.type === card.type)
        );
    } else {
        selectedCards.value.push(card);
    }
};

const selectedLights = computed(() =>
    selectedCards.value.filter((card) => card.type === CardType.LIGHT)
);
const selectedSeeds = computed(() =>
    selectedCards.value.filter((card) => card.type === CardType.SEED)
);
const selectedRibbons = computed(() =>
    selectedCards.value.filter((card) => card.type === CardType.RIBBON)
);

const resetCards = () => {
    selectedCards.value = [];
    chaffCount.value = 0;
};

const includeHanamiTsukimi = ref(true);
const doubleOver7 = ref(true);
const opponentKoikoi = ref(false);

const calculatedScore = computed(() => {
    const yaku: Yaku = {};
    let totalScore = 0;

    if (selectedLights.value.length === 5) {
        yaku["五光"] = 10;
    } else if (selectedLights.value.length === 4) {
        if (selectedLights.value.some((card) => card.month === 11)) {
            yaku["雨入り四光"] = 7;
        } else {
            yaku["四光"] = 8;
        }
    } else if (
        selectedLights.value.length === 3 &&
        !selectedLights.value.some((card) => card.month === 11)
    ) {
        yaku["三光"] = 5;
    }

    if (includeHanamiTsukimi.value) {
        if (
            selectedLights.value.some((card) => card.month === 3) &&
            selectedSeeds.value.some((card) => card.month === 9)
        ) {
            yaku["花見で一杯"] = 5;
        }
        if (
            selectedLights.value.some((card) => card.month === 8) &&
            selectedSeeds.value.some((card) => card.month === 9)
        ) {
            yaku["月見で一杯"] = 5;
        }
    }

    const animals = selectedSeeds.value.filter((card) =>
        [6, 7, 10].includes(card.month)
    );
    if (animals.length === 3) {
        const additionalSeeds = selectedSeeds.value.length - 3;
        yaku["猪鹿蝶"] = { base: 5, additional: additionalSeeds };
    }

    if (selectedSeeds.value.length >= 5) {
        const additionalSeeds = selectedSeeds.value.length - 5;
        yaku["タネ"] = { base: 1, additional: additionalSeeds };
    }

    const redRibbons = selectedRibbons.value.filter((card) =>
        [1, 2, 3].includes(card.month)
    );
    const blueRibbons = selectedRibbons.value.filter((card) =>
        [6, 9, 10].includes(card.month)
    );
    const otherRibbons = selectedRibbons.value.filter(
        (card) => ![1, 2, 3, 6, 9, 10].includes(card.month)
    );

    if (redRibbons.length === 3 && blueRibbons.length === 3) {
        yaku["赤短・青短の重複"] = {
            base: 10,
            additional: otherRibbons.length,
        };
    } else {
        if (redRibbons.length === 3) {
            yaku["赤短"] = {
                base: 5,
                additional: selectedRibbons.value.length - 3,
            };
        }
        if (blueRibbons.length === 3) {
            yaku["青短"] = {
                base: 5,
                additional: selectedRibbons.value.length - 3,
            };
        }
    }

    if (selectedRibbons.value.length >= 5) {
        const additionalRibbons = selectedRibbons.value.length - 5;
        yaku["タン"] = { base: 1, additional: additionalRibbons };
    }

    if (chaffCount.value >= 10) {
        const additionalChaff = chaffCount.value - 10;
        yaku["カス"] = { base: 1, additional: additionalChaff };
    }

    totalScore = Object.values(yaku).reduce((sum, score) => {
        if (typeof score === "number") {
            return sum + score;
        } else {
            return sum + score.base + score.additional;
        }
    }, 0);

    if (doubleOver7.value && totalScore >= 7) {
        totalScore *= 2;
    }

    if (opponentKoikoi.value) {
        totalScore *= 2;
    }

    return { totalScore, yaku };
});

const svgIcons = ref<{ [key: string]: string }>({});

onMounted(async () => {
    for (const card of [
        ...availableLights,
        ...availableSeeds,
        ...availableRibbons,
    ]) {
        if (card.icon) {
            try {
                // JSONで指定されたパスを使用してSVGファイルを動的にインポート
                const iconModule = await import(`../${card.icon}`);
                svgIcons.value[card.icon] = iconModule.default;
            } catch (error) {
                console.error(`Failed to load SVG for ${card.icon}:`, error);
            }
        }
    }
});
</script>

<template>
    <div class="koikoi-calculator">
        <header class="header">
            <div class="header-content">
                <div class="current-score">
                    現在の得点:
                    <span class="score">{{ calculatedScore.totalScore }}</span>
                </div>
            </div>
        </header>
        <h1>花札（こいこい）得点計算</h1>

        <h2>獲得したカードを選択</h2>
        <div class="card-selection">
            <div
                v-for="(cards, type) in {
                    lights: availableLights,
                    seeds: availableSeeds,
                    ribbons: availableRibbons,
                }"
                :key="type"
                class="card-group"
            >
                <h3>
                    {{
                        type === "lights"
                            ? "光札"
                            : type === "seeds"
                            ? "種札"
                            : "短冊札"
                    }}
                </h3>
                <div
                    v-for="card in cards"
                    :key="`${card.month}-${card.type}`"
                    class="card-item"
                >
                    <label>
                        <input
                            type="checkbox"
                            :checked="isSelected(card)"
                            @change="toggleCard(card)"
                        />
                        <img
                            v-if="card.icon && svgIcons[card.icon]"
                            :src="svgIcons[card.icon]"
                            class="card-icon"
                            :alt="card.name"
                        />
                        {{ card.name }}
                    </label>
                </div>
            </div>

            <div class="card-group">
                <h3>カス札</h3>
                <div class="card-item">
                    <label>
                        カス札の枚数:
                        <input
                            inputmode="numeric"
                            type="number"
                            v-model="chaffCount"
                            min="0"
                            max="20"
                        />
                    </label>
                </div>
            </div>
        </div>

        <button @click="resetCards" class="reset-button">札をリセット</button>

        <div class="options">
            <label>
                <input type="checkbox" v-model="includeHanamiTsukimi" />
                花見で一杯・月見で一杯を含める
            </label>
            <label>
                <input type="checkbox" v-model="doubleOver7" />
                7点以上で2倍
            </label>
            <label>
                <input type="checkbox" v-model="opponentKoikoi" />
                相手がこいこいした（2倍）
            </label>
        </div>

        <div class="score-result">
            <h2>得点計算結果</h2>
            <p>総得点: {{ calculatedScore.totalScore }}</p>
            <h3>獲得した役:</h3>
            <ul>
                <li v-for="(score, role) in calculatedScore.yaku" :key="role">
                    {{ role }}:
                    <template v-if="typeof score === 'number'">
                        {{ score }}点
                    </template>
                    <template v-else>
                        {{ score.base }}点
                        <template v-if="score.additional > 0">
                            +{{ score.additional }}点
                        </template>
                    </template>
                </li>
            </ul>
            <p v-if="doubleOver7 && calculatedScore.totalScore >= 7">
                7点以上のため2倍適用
            </p>
            <p v-if="opponentKoikoi">相手こいこいのため2倍適用</p>
        </div>
    </div>
</template>

<style scoped>
.koikoi-calculator {
    font-family: Arial, sans-serif;
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
}

.card-selection {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 20px;
    margin-bottom: 20px;
}

.card-group {
    background-color: #f0f0f0;
    padding: 15px;
    border-radius: 5px;
}

.card-item {
    margin-bottom: 10px;
}

.options {
    margin-bottom: 20px;
}

.calculate-button {
    display: block;
    width: 100%;
    padding: 10px;
    font-size: 16px;
    background-color: #4caf50;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    margin-bottom: 20px;
}

.calculate-button:hover {
    background-color: #45a049;
}

.score-result {
    background-color: #e9e9e9;
    padding: 20px;
    border-radius: 5px;
}

h1,
h2,
h3 {
    color: #333;
}

ul {
    list-style-type: none;
    padding: 0;
}

li {
    margin-bottom: 5px;
}

input[type="number"] {
    width: 60px;
}

.options label {
    display: block;
    margin-bottom: 10px;
}

.reset-button {
    display: block;
    width: 100%;
    padding: 10px;
    font-size: 16px;
    background-color: #f44336;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    margin-bottom: 20px;
}

.reset-button:hover {
    background-color: #d32f2f;
}

.header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    background-color: white;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    z-index: 1000;
}

.header-content {
    max-width: 800px;
    margin: 0 auto;
    padding: 10px 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.header h1 {
    margin: 0;
    font-size: 24px;
}

.current-score {
    font-size: 18px;
    font-weight: bold;
}

.score {
    color: #4caf50;
}

.main-content {
    padding-top: 80px;
    padding-left: 20px;
    padding-right: 20px;
}

.card-icon {
    display: inline-block;
    width: 30px;
    height: 30px;
    margin-right: 5px;
    vertical-align: middle;
}

.card-icon svg {
    width: 100%;
    height: 100%;
    display: block; /* これを追加 */
}
</style>
