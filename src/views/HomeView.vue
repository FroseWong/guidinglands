<template>
  <div class="home">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <div class="title">聚魔之地魔物紀錄</div>
    <div class="search-text">
      搜尋中的文字：<span>{{ searchingValue }}</span>
    </div>
    <div class="search-div">
      <input type="text" class="filter-input" v-model="searchValue" />
      <font-awesome-icon icon="fa-solid fa-xmark" @click="inputXBtnClick" />
      <!-- <div class="only-no-hit-div">
        <input type="checkbox" />
        <div class="only-no-hit">只顯示沒有打過的魔物</div>
      </div> -->
      <div class="unone search-btn" @click="searchBtnClick">搜尋</div>
      <div
        class="unone onlyhit-btn"
        :class="{ on: onlyhitStatus }"
        @click="onlyhitBtnClick"
      >
        只顯示沒有打過的魔物
      </div>

      <!-- <div class="unone search-btn" @click="allRecover">恢復原狀</div> -->
      <div class="unone search-btn" @click="hitRecover">打過狀態清空</div>
      <div class="unone search-btn" @click="saveData">儲存</div>
      <div class="unone search-btn" @click="deleteSave">刪除存檔</div>
    </div>
    <div class="slider-check">
      <input type="checkbox" id="use-slider" v-model="useSlider" />
      <label for="use-slider">是否啟用slider filter</label>
    </div>
    <div class="slider-space" v-show="useSlider">
      <div class="each-slider" v-for="eachSlider in sliderData">
        <div class="slider-title">{{ eachSlider.zoneName }}</div>

        <input
          class="slider-input"
          type="range"
          id="volume"
          name="volume"
          min="0"
          max="7"
          v-model="eachSlider.zoneLv"
        />
        <div class="slider-num">
          {{ eachSlider.zoneLv === "0" ? "" : eachSlider.zoneLv }}
        </div>
      </div>
    </div>

    <div class="show-space">
      <div
        class="each-data"
        v-for="eachData in showMonsterData"
        v-show="shouldShow(eachData.monsters)"
      >
        <!-- <div class="left-div">
          <div class="check-div">
            <input type="checkbox" />
          </div>
        </div> -->
        <!-- <div class="right-div"> -->
        <div class="zone-div">
          <div class="zone-title">地區：</div>
          <div class="zone">{{ eachData.zone }}</div>
        </div>
        <div class="lv-div">
          <div class="lv-title">等級：</div>
          <div class="lv">{{ eachData.lv }}</div>
        </div>
        <div class="star-div">
          <div class="star-title">星星：</div>
          <div class="star">
            <font-awesome-icon
              icon="fa-solid fa-star"
              v-for="x in eachData.star"
            />
            <!-- {{ eachData.star }} -->
          </div>
        </div>
        <div
          class="name-div"
          v-for="eachMonster in eachData.monsters"
          @click="clickMonster(eachMonster.name)"
        >
          <!-- <input
            type="checkbox"
            class="check-status"
            v-model="eachMonster.status"
          /> -->
          <font-awesome-icon
            icon="fa-solid fa-circle-check"
            class="fa-circle-check"
            :class="{ hide: !eachMonster.status }"
          />
          <!-- <div class="name-title">名稱：</div> -->
          <div class="name">{{ eachMonster.name }}</div>
        </div>
        <!-- </div> -->
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch, nextTick } from "vue";
import guidingLandsData from ".././data/monster_data_full.js";

const monsterData = ref(); // 完整的資料
const showMonsterData = ref(); // 用來顯示的
const searchValue = ref("");
const searchingValue = ref(""); // 正在搜尋的文字
const onlyhitStatus = ref(false);

const useSlider = ref(false);
const sliderData = ref([
  {
    zoneName: "森林地帶",
    zoneLv: "0"
  },
  {
    zoneName: "荒地地帶",
    zoneLv: "0"
  },
  {
    zoneName: "陸珊瑚地帶",
    zoneLv: "0"
  },
  {
    zoneName: "瘴氣地帶",
    zoneLv: "0"
  },
  {
    zoneName: "熔岩地帶",
    zoneLv: "0"
  },
  {
    zoneName: "冰雪地帶",
    zoneLv: "0"
  }
]);

onMounted(() => {
  if (localStorage.getItem("guidingLandsData"))
    monsterData.value = JSON.parse(localStorage.getItem("guidingLandsData"));
  else monsterData.value = guidingLandsData;
  clone();
});

const sliderFilter = () => {
  const sliderFilterArr = sliderData.value.filter(
    (eachData) => eachData.zoneLv != "0"
  );

  if (sliderFilterArr.length === 0) return;

  // console.log("sliderFilterArr", sliderFilterArr);

  // console.log("sliderFilter");

  // console.log("showMonsterData", showMonsterData.value);

  showMonsterData.value = showMonsterData.value.filter((eachSpace) => {
    let check = false;
    sliderFilterArr.forEach((eachFilter) => {
      // console.log("eachSpace.zone", eachSpace.zone);
      // console.log("eachFilter.zoneName", eachFilter.zoneName);

      if (eachSpace.zone === eachFilter.zoneName) {
        if (eachSpace.lv == eachFilter.zoneLv) check = true;
      }
    });
    return check;
  });

  // console.log("filter完畢", showMonsterData.value);
};

const letsSearch = () => {
  filterMonster();
  searchMonster();
  if (useSlider.value) sliderFilter();
};

const clickMonster = (name) => {
  monsterData.value.forEach((eachData) => {
    // console.log("eachData", eachData);
    eachData.monsters.forEach((eachMonster) => {
      if (eachMonster.name === name) eachMonster.status = !eachMonster.status;
    });
  });
  // searchMonster();
  letsSearch();
};

const filterMonster = () => {
  clone();

  if (onlyhitStatus.value) {
    showMonsterData.value.forEach((eachData) => {
      eachData.monsters = eachData.monsters.filter(
        (eachMonster) => !eachMonster.status
      );
    });
  }
};
//  只顯示沒有打過的魔物
const onlyhitBtnClick = () => {
  onlyhitStatus.value = !onlyhitStatus.value;
  // clone();

  letsSearch();
  // console.log("monsterData.value", monsterData.value);
};

// console.log("guidingLandsData", guidingLandsData);

const clone = () => {
  showMonsterData.value = JSON.parse(JSON.stringify(monsterData.value));
};

// 沒有monster的話 不顯示
const shouldShow = (monsters) => {
  // console.log("monsters", monsters);
  const filteredMonster = monsters.filter((eachMonster) => !eachMonster.status);

  // console.log("filteredMonster", filteredMonster);

  return onlyhitStatus.value ? filteredMonster.length > 0 : monsters.length > 0;
};

// 所有魔物恢復原狀
const hitRecover = () => {
  const confirmed = confirm("你確定要打過狀態清空嗎");

  if (confirmed) {
    monsterData.value.forEach((eachData) =>
      eachData.monsters.forEach((eachMonster) => (eachMonster.status = false))
    );

    letsSearch();
  }
};

// 搜尋狀態恢復
const allRecover = () => {
  searchValue.value = "";
  searchingValue.value = "";
  onlyhitStatus.value = false;

  // hitRecover();
};

const saveData = () => {
  localStorage.setItem("guidingLandsData", JSON.stringify(monsterData.value));
  alert("儲存成功");
};

const searchBtnClick = () => {
  searchingValue.value = searchValue.value.trim();
  letsSearch();
};

const searchMonster = () => {
  // console.log("showMonsterData.value!!", showMonsterData.value);
  showMonsterData.value.forEach((eachData) => {
    eachData.monsters = eachData.monsters.filter((eachMonster) => {
      return eachMonster.name.includes(searchingValue.value.trim());
    });
  });
};

const deleteSave = () => {
  const confirmed = confirm("你確定要刪除存檔嗎");

  if (confirmed) {
    localStorage.removeItem("guidingLandsData");
    alert("刪除成功");
  }
};

const inputXBtnClick = () => {
  searchValue.value = "";
  // searchingValue.value = "";
};
</script>

<style lang="scss"></style>
