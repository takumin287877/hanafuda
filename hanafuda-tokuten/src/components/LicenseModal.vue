<template>
    <div class="modal-overlay" @click="$emit('close')">
        <div class="modal-content" @click.stop>
            <h2>ライセンス情報</h2>
            <!-- ここにライセンス情報を記述 -->
            <p>
                花札の画像は以下のライセンスの下で提供されています。<br />
                <hr>
                光札 <br /><br />User:Louiemantia, すけじょ, すけじょ -
                File:Hanafuda January Hikari.svg, 投稿者自身による著作物, CC
                表示-継承 4.0,<br />

                https://commons.wikimedia.org/w/index.php?curid=108973794による
                User:Louiemantia, すけじょ - File:Hanafuda March Hikari.svg,
                投稿者自身による著作物, CC 表示-継承 4.0,<br />

                https://commons.wikimedia.org/w/index.php?curid=108973825による
                Louie Mantia, すけじょ - File:Hanafuda August Hikari.svg,
                投稿者自身による著作物, CC 表示-継承 4.0,<br />

                https://commons.wikimedia.org/w/index.php?curid=108973763による
                Louie Mantia, すけじょ - File:Hanafuda November Hikari.svg,
                投稿者自身による著作物, CC 表示-継承 4.0,<br />

                https://commons.wikimedia.org/w/index.php?curid=108973842による
                https://commons.wikimedia.org/w/index.php?curid=108973780
                <hr>タネ札　猪鹿蝶 <br /><br />
                Louie Mantia, すけじょ -File:Hanafuda June Tane.svg,
                投稿者自身による著作物, CC 表示-継承 4.0,<br />
                https://commons.wikimedia.org/w/index.php?curid=108973822による
                Louie Mantia, すけじょ - File:Hanafuda July Tane.svg,
                投稿者自身による著作物, CC 表示-継承 4.0,<br />
                https://commons.wikimedia.org/w/index.php?curid=108973811による
                Louie Mantia, すけじょ - File:Hanafuda October Tane.svg,
                投稿者自身による著作物, CC 表示-継承 4.0,<br />
                https://commons.wikimedia.org/w/index.php?curid=108973861による
                <br /><br />盃 <br />
                Louie Mantia, すけじょ - File:Hanafuda September Tane.svg,
                投稿者自身による著作物, CC 表示-継承 4.0,<br />
                https://commons.wikimedia.org/w/index.php?curid=108973870による
                <hr>短冊札　赤短 <br /><br />
                User:Louiemantia, すけじょ -
                File:Hanafuda January Tanzaku.svg, 投稿者自身による著作物, CC
                表示-継承 4.0,<br />
                https://commons.wikimedia.org/w/index.php?curid=108973799による
                User:Louiemantia, すけじょ - , 投稿者自身による著作物, CC
                表示-継承 4.0,<br />
                https://commons.wikimedia.org/w/index.php?curid=108973793による
                User:Louiemantia, すけじょ - File:Hanafuda March Tanzaku.svg,
                投稿者自身による著作物, CC 表示-継承 4.0,<br />
                https://commons.wikimedia.org/w/index.php?curid=108973833による
                <br />
                <br />青短<br />
                Louie Mantia, すけじょ - File:Hanafuda June Tanzaku.svg,
                投稿者自身による著作物, CC 表示-継承 4.0,<br />
                https://commons.wikimedia.org/w/index.php?curid=108973823による
                Louie Mantia, すけじょ - File:Hanafuda September Tanzaku.svg,
                投稿者自身による著作物, CC 表示-継承 4.0,<br />
                https://commons.wikimedia.org/w/index.php?curid=108973874による
                Louie Mantia, すけじょ - File:Hanafuda October Tanzaku.svg,
                投稿者自身による著作物, CC 表示-継承 4.0,<br />
                https://commons.wikimedia.org/w/index.php?curid=108973863による
                <hr>
            </p>
            <!-- ライセンス情報を追加 -->
            <button @click="$emit('close')" class="close-button">閉じる</button>
        </div>
    </div>
</template>

<script setup lang="ts">
// 必要に応じてスクリプトを追加
</script>

<style scoped>
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
}

.modal-content {
    background-color: white;
    padding: 20px;
    border-radius: 5px;
    max-width: 80%;
    max-height: 80%;
    overflow-y: auto;
}

.close-button {
    margin-top: 20px;
    padding: 10px 20px;
    background-color: #f44336;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.close-button:hover {
    background-color: #d32f2f;
}
</style>
