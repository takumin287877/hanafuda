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
        <div class="main-content">
            <div class="left-column">
                <h1>花札（こいこい）得点計算</h1>

                <h2>獲得した札を選択</h2>
                <div class="card-selection">
                    <!-- 光札の選択 -->
                    <div class="card-group fit-size">
                        <h3>光札</h3>
                        <div
                            v-for="card in availableLights"
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

                    <!-- タネ札の選択 -->
                    <div class="card-group fit-size">
                        <h3>タネ札</h3>
                        <!-- 特殊なタネ札 -->
                        <div
                            v-for="card in specialSeeds"
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
                        <!-- その他のタネ札 -->
                        <div class="card-item">
                            <label>
                                その他のタネ札の枚数:
                                <select v-model="otherSeedsCount">
                                    <option
                                        v-for="n in 6"
                                        :key="n - 1"
                                        :value="n - 1"
                                    >
                                        {{ n - 1 }}
                                    </option>
                                </select>
                            </label>
                            <p>合計: {{ totalSeedsCount }}枚</p>
                        </div>
                    </div>

                    <!-- タン札の選択 -->
                    <div class="card-group fit-size">
                        <h3>短冊札</h3>
                        <!-- 赤短と青短 -->
                        <div
                            v-for="card in specialRibbons"
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
                        <!-- その他のタン札 -->
                        <div class="card-item">
                            <label>
                                その他の短冊札の枚数:
                                <select v-model="otherRibbonsCount">
                                    <option
                                        v-for="n in 5"
                                        :key="n - 1"
                                        :value="n - 1"
                                    >
                                        {{ n - 1 }}
                                    </option>
                                </select>
                                <p>合計: {{ totalRibbonsCount }}枚</p>
                            </label>
                        </div>
                    </div>

                    <!-- カス札の選択 -->
                    <div class="card-group">
                        <h3>カス札</h3>
                        <div class="card-item">
                            <label>
                                カス札の枚数:
                                <select v-model="chaffCount">
                                    <option
                                        v-for="n in 25"
                                        :key="n - 1"
                                        :value="n - 1"
                                    >
                                        {{ n - 1 }}
                                    </option>
                                </select>
                            </label>
                        </div>
                    </div>
                </div>
                <br />
                <!-- こいこいの選択を追加 -->
                <div class="card-selection">
                    <div class="card-group">
                        <h3>こいこい</h3>
                        <div class="koikoi-toggle">
                            <span>相手がこいこい</span>
                            <button
                                @click="opponentKoikoi = !opponentKoikoi"
                                :class="{ active: opponentKoikoi }"
                            >
                                {{ opponentKoikoi ? "した" : "してない" }}
                            </button>
                        </div>
                    </div>
                    <div class="card-group">
                        <div class="options">
                            <label>
                                <h3>オプション</h3>
                                <input
                                    type="checkbox"
                                    v-model="includeHanamiTsukimi"
                                />
                                花見で一杯・月見で一杯を含める
                            </label>
                            <label>
                                <input type="checkbox" v-model="doubleOver7" />
                                7点以上で2倍
                            </label>
                        </div>
                    </div>
                </div>
            </div>
            <hr class="options-hr" />
            <div class="right-column">
                <div class="score-result">
                    <h2>得点計算結果</h2>
                    <p>総得点: {{ calculatedScore.totalScore }}</p>
                    <hr />
                    <h3>役:</h3>
                    <ul>
                        <li
                            v-for="(score, role) in calculatedScore.yaku"
                            :key="role"
                        >
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
                    <hr />
                    <p v-if="doubleOver7 && calculatedScore.totalScore >= 7">
                        ※7点以上のため2倍適用
                    </p>
                    <p v-if="opponentKoikoi">※相手こいこいのため2倍適用</p>
                </div>
                <button @click="resetCards" class="reset-button">
                    札をリセット
                </button>
                <license-modal
                    v-if="showLicenseModal"
                    @close="showLicenseModal = false"
                />
                <button @click="showLicenseModal = true" class="license-button">
                    ライセンス
                </button>
            </div>
        </div>

        <!-- モーダルウィンドウ -->
        <div v-if="showModal" class="modal">
            <div class="modal-content">
                <p>本当に札をリセットしますか？</p>
                <button class="reset-button-yes" @click="confirmReset">
                    はい
                </button>
                <button class="reset-button-no" @click="showModal = false">
                    いいえ
                </button>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import LicenseModal from "./LicenseModal.vue";

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

interface Score {
    base: number;
    additional: number;
}

const selectedCards = ref<Card[]>([]);
// カス札の枚数
const chaffCount = ref(6);
// その他のタネ札の枚数
const otherSeedsCount = ref(0);
// その他の短冊札の枚数
const otherRibbonsCount = ref(0);

const availableLights: Card[] = [
    {
        month: 1,
        type: CardType.LIGHT,
        name: "松に鶴",
        icon: "./assets/札/光札/Hanafuda_January_Hikari_Alt.svg",
    },
    {
        month: 3,
        type: CardType.LIGHT,
        name: "桜に幔幕（花）",
        icon: "./assets/札/光札/Hanafuda_March_Hikari_Alt.svg",
    },
    {
        month: 8,
        type: CardType.LIGHT,
        name: "芒に望月　（月）",
        icon: "./assets/札/光札/Hanafuda_August_Hikari_Alt.svg",
    },
    {
        month: 12,
        type: CardType.LIGHT,
        name: "桐に鳳凰",
        icon: "./assets/札/光札/Hanafuda_December_Hikari_Alt.svg",
    },
    {
        month: 11,
        type: CardType.LIGHT,
        name: "柳に小野道風 （雨）",
        icon: "./assets/札/光札/Hanafuda_November_Hikari_Alt.svg",
    },
];
const specialSeeds: Card[] = [
    {
        month: 9,
        type: CardType.SEED,
        name: "菊に盃 (一杯)",
        icon: "./assets/札/タネ札/Hanafuda_September_Tane_Alt.svg",
    },
    {
        month: 7,
        type: CardType.SEED,
        name: "萩に猪",
        icon: "./assets/札/タネ札/Hanafuda_July_Tane_Alt.svg",
    },
    {
        month: 10,
        type: CardType.SEED,
        name: "紅葉に鹿",
        icon: "./assets/札/タネ札/Hanafuda_October_Tane_Alt.svg",
    },
    {
        month: 6,
        type: CardType.SEED,
        name: "牡丹に蝶",
        icon: "./assets/札/タネ札/Hanafuda_June_Tane_Alt.svg",
    },
];

const specialRibbons: Card[] = [
    {
        month: 1,
        type: CardType.RIBBON,
        name: "松の赤短",
        icon: "./assets/札/タン札/Hanafuda_January_Tanzaku_Alt.svg",
    },
    {
        month: 2,
        type: CardType.RIBBON,
        name: "梅の赤短",
        icon: "./assets/札/タン札/Hanafuda_February_Tanzaku_Alt.svg",
    },
    {
        month: 3,
        type: CardType.RIBBON,
        name: "桜の赤短",
        icon: "./assets/札/タン札/Hanafuda_March_Tanzaku_Alt.svg",
    },
    {
        month: 6,
        type: CardType.RIBBON,
        name: "牡丹の青短",
        icon: "./assets/札/タン札/Hanafuda_June_Tanzaku_Alt.svg",
    },
    {
        month: 9,
        type: CardType.RIBBON,
        name: "菊の青短",
        icon: "./assets/札/タン札/Hanafuda_September_Tanzaku_Alt.svg",
    },
    {
        month: 10,
        type: CardType.RIBBON,
        name: "紅葉の青短",
        icon: "./assets/札/タ札/Hanafuda_October_Tanzaku_Alt.svg",
    },
];

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

const totalSeedsCount = computed(
    () => selectedSeeds.value.length + otherSeedsCount.value
);
const totalRibbonsCount = computed(
    () => selectedRibbons.value.length + otherRibbonsCount.value
);

const showModal = ref(false);

const resetCards = () => {
    showModal.value = true; // モーダルを表示
};

const confirmReset = () => {
    selectedCards.value = [];
    chaffCount.value = 6;
    otherSeedsCount.value = 0;
    otherRibbonsCount.value = 0;
    opponentKoikoi.value = false;
    showModal.value = false; // モーダルを閉じる
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
        const additionalSeeds = totalSeedsCount.value - 3;
        yaku["猪鹿蝶"] = { base: 5, additional: additionalSeeds };
    }

    if (totalSeedsCount.value >= 5) {
        const additionalSeeds = totalSeedsCount.value - 5;
        yaku["タネ"] = { base: 1, additional: additionalSeeds };
    }

    const redRibbons = selectedRibbons.value.filter((card) =>
        [1, 2, 3].includes(card.month)
    );
    const blueRibbons = selectedRibbons.value.filter((card) =>
        [6, 9, 10].includes(card.month)
    );

    if (redRibbons.length === 3 && blueRibbons.length === 3) {
        yaku["赤短・青短の重複"] = {
            base: 10,
            additional: totalRibbonsCount.value - 6,
        };
    } else {
        if (redRibbons.length === 3) {
            yaku["赤短"] = {
                base: 5,
                additional: totalRibbonsCount.value - 3,
            };
        }
        if (blueRibbons.length === 3) {
            yaku["青短"] = {
                base: 5,
                additional: totalRibbonsCount.value - 3,
            };
        }
    }

    if (totalRibbonsCount.value >= 5) {
        const additionalRibbons = totalRibbonsCount.value - 5;
        yaku["タン"] = { base: 1, additional: additionalRibbons };
    }

    if (chaffCount.value >= 10) {
        const additionalChaff = chaffCount.value - 10;
        yaku["カス"] = { base: 1, additional: additionalChaff };
    }

    totalScore = Object.values(yaku).reduce((sum: number, score: number | Score) => {
        if (typeof score === 'number') {
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
        ...specialSeeds,
        ...specialRibbons,
    ]) {
        if (card.icon) {
            try {
                /* @vite-ignore */
                const iconModule = await import(`../${card.icon}`);
                svgIcons.value[card.icon] = iconModule.default;
            } catch (error) {
                console.error(`Failed to load SVG for ${card.icon}:`, error);
            }
        }
    }
});

// ライセンスモーダルの表示状態
const showLicenseModal = ref(false);
</script>

<style scoped>
.koikoi-calculator {
    font-family: Arial, sans-serif;
    max-width: 85%;
    margin: 0 auto;
    padding: 20px;
}

.card-selection {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 20px;
    /* margin-bottom: 20px; */
}

.card-group {
    background-color: #f0f0f0;
    padding: 15px;
    border-radius: 5px;
}

.fit-size {
    height: auto;
    display: flex;
    flex-direction: column;
}

.card-item {
    margin-bottom: 10px;
}

.options {
    margin-bottom: 20px;
}

.calculate-button {
    display: block;
    width: 100px;
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
.header {
    display: none;
}

.score-result {
    margin-top: 72px;
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
    margin-top: 20px;
}

.reset-button:hover {
    background-color: #d32f2f;
}

.current-score {
    font-size: 18px;
    font-weight: bold;
}

.score {
    color: #4caf50;
}

.main-content {
    display: flex;
    gap: 20px;
    /* padding-top: 80px; */
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

.koikoi-toggle {
    display: flex;
    align-items: center;
    justify-content: space-between;
}

.koikoi-toggle button {
    padding: 10px 20px;
    font-size: 16px;
    border-radius: 5px;
    cursor: pointer;
    background-color: #ccc;
    color: #333;
    border-color: #000;
    transition: background-color 0.3s, color 0.3s;
    width: 120px;
    height: 40px;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
}

.koikoi-toggle button.active {
    background-color: #4caf50;
    color: white;
    border: none;
}

@media (max-width: 768px) {
    .koikoi-toggle button {
        padding: 8px 16px;
        font-size: 14px;
        width: 100px;
        height: 36px;
    }
}

.modal {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
}

.modal-content {
    background: white;
    padding: 20px;
    border-radius: 5px;
    text-align: center;
    width: 300px;
    height: 130px;
}

.reset-button-yes {
    background-color: #4caf50;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    margin-bottom: 20px;
    width: 100px;
    height: 50px;
    margin-right: 10px;
    margin-top: 10px;
}

.reset-button-no {
    background-color: #f44336;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    margin-bottom: 20px;
    width: 100px;
    height: 50px;
    margin-left: 10px;
    margin-top: 10px;
}

.left-column {
    flex: 1;
    max-width: 70%;
}

.right-column {
    flex: 0 0 30%;
    position: sticky;
    top: 100px;
    align-self: flex-start;
}

.license-button {
    display: block;
    width: 100%;
    padding: 10px;
    font-size: 16px;
    background-color: rgba(7, 59, 80, 0.8);
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.license-button:hover {
    background-color: rgba(7, 90, 122, 1);
}

.options-hr {
    color: #000000;
}

select {
    appearance: none;
    -webkit-appearance: none;
    -moz-appearance: none;
    background-color: #f8f8f8;
    border: 1px solid #ddd;
    border-radius: 4px;
    padding: 8px 30px 8px 10px;
    font-size: 14px;
    color: #333;
    cursor: pointer;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath fill='%23333' d='M10.293 3.293L6 7.586 1.707 3.293A1 1 0 00.293 4.707l5 5a1 1 0 001.414 0l5-5a1 1 0 10-1.414-1.414z'/%3E%3C/svg%3E");
    background-repeat: no-repeat;
    background-position: right 10px center;
    background-size: 12px;
}

select:focus {
    outline: none;
    border-color: #4caf50;
    box-shadow: 0 0 0 2px rgba(76, 175, 80, 0.2);
}

select:hover {
    background-color: #f0f0f0;
}

@media (max-width: 1024px) {
    .main-content {
        flex-direction: column;
    }

    .left-column {
        flex: 1;
        max-width: 100%;
    }

    .right-column {
        position: static;
        width: 100%;
    }
    .score-result {
        margin-top: 10px;
    }

    .header {
        display: block;
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
    .options-hr {
        margin-top: 15px;
        width: 110%;
        margin-left: -5%;
    }
}
</style>
