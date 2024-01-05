<template>
  <div>
    <h2>길드원 본캐 찾기</h2>
    <form @submit.prevent="submitForm1">
      <label for="guildname">길드 이름:</label>
      <input v-model="guildname" type="text" id="guildname" name="guildname">
      <label for="worldname">서버명:</label>
      <select v-model="worldname" id="worldname" name="worldname">
        <option value="스카니아">스카니아</option>
        <option value="베라">베라</option>
        <option value="루나">루나</option>
        <option value="제니스">제니스</option>
        <option value="크로아">크로아</option>
        <option value="유니온">유니온</option>
        <option value="엘리시움">엘리시움</option>
        <option value="이노시스">이노시스</option>
        <option value="레드">레드</option>
        <option value="오로라">오로라</option>
        <option value="아케인">아케인</option>
        <option value="노바">노바</option>
        <option value="리부트">리부트</option>
        <option value="리부트2">리부트2</option>
        <option value="버닝">버닝</option>
        <option value="버닝2">버닝2</option>
        <option value="버닝3">버닝3</option>
      </select>
      <button type="submit">확인</button>
    </form>

    <div v-if="ocidDataGuild">
      <table class="center-table">
        <thead>
          <tr>
            <th class="header-col">순번</th>
            <th class="header-col">길드 멤버</th>
            <th class="header-col">본캐</th>
            <th class="header-col">본캐 길드</th>
            <th class="header-col">순번</th>
            <th class="header-col">길드 멤버</th>
            <th class="header-col">본캐</th>
            <th class="header-col">본캐 길드</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(member, index) in guildBasicData.guild_member.slice(0, 100)" :key="index"
            :class="{ 'no-data-row': show[index] === '데이터 없음' || show3[index] === '데이터 없음' }" class="member-container">
            <td>{{ index + 1 }}</td>
            <td>{{ guildBasicData.guild_member[index] }}</td>
            <td :class="{ 'highlight-col': guildname !== show3[index] }">{{ show[index] || '데이터 없음' }}</td>
            <td :class="{ 'highlight-col': guildname !== show3[index] }">{{ show3[index] || '데이터 없음' }}</td>
            <!-- 다음 4개의 열을 추가합니다 -->
            <td>{{ index + 101 }}</td>
            <td>{{ guildBasicData.guild_member[index + 100] }}</td>
            <td :class="{ 'highlight-col': guildname !== show3[index + 100] }">{{ show[index + 100] || '데이터 없음' }}</td>
            <td :class="{ 'highlight-col': guildname !== show3[index + 100] }">{{ show3[index + 100] || '데이터 없음' }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>


<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';

const API = "https://open.api.nexon.com/maplestory/v1/";
const API_KEY = "live_9961b9606a2eeae06255293a5e7e010a06b1f76134f579fa3769d94e454938deed87e563828f82c2158069678d13312b";

const guildname = ref('');
const worldname = ref('');
const ocidDataGuild = ref('');
const guildBasicData = ref('');
const guildMember = ref(['']);
const guildMemberOcid = ref(['']);
const guildMemberMain = ref(['']);
const guildMemberMainOcid = ref(['']);
const guildMemberMainGuild = ref(['']);
const show = ref(['']);
const show3 = ref(['']);

const currentDate = ref(getCurrentDate());

function getCurrentDate() {
  const today = new Date();
  // 하루를 빼줌
  today.setDate(today.getDate() - 1);

  // YYYY-MM-DD 형식으로 포맷팅
  const year = today.getFullYear();
  const month = String(today.getMonth() + 1).padStart(2, '0');
  const day = String(today.getDate()).padStart(2, '0');

  return `${year}-${month}-${day}`;
}

function resetTable() {
  ocidDataGuild.value = '';
  guildBasicData.value = '';
  guildMember.value = [];
  show.value = [];
  show3.value = [];
}

async function submitForm1() {
  resetTable();
  await getGuildOcid();
  await getGuildBasic();
}


async function getGuildOcid() {
  try {
    const guildOcid = await axios.get(`${API}guild/id?guild_name=${guildname.value}&world_name=${worldname.value}`, {
      headers: {
        'x-nxopen-api-key': API_KEY,
        'accept': 'application/json',
      },
    });
    ocidDataGuild.value = guildOcid.data;
  } catch (error) {
    console.error("Error fetching userOcid:", error.response.data);
  }
}

async function getGuildBasic() {
  try {
    const guildBasic = await axios.get(`${API}guild/basic?oguild_id=${ocidDataGuild.value.oguild_id}&date=${currentDate.value}`, {
      headers: {
        'x-nxopen-api-key': API_KEY,
        'accept': 'application/json',
      },
    });
    guildBasicData.value = guildBasic.data;
    for (let i = 0; i < guildBasicData.value.guild_member_count; i++) {
      guildMember.value[i] = guildBasicData.value.guild_member[i];
      try {
        guildMemberOcid.value[i] = await axios.get(`${API}id?character_name=${guildMember.value[i]}`, {
          headers: {
            'x-nxopen-api-key': API_KEY,
            'accept': 'application/json',
          },
        });
      } catch (error) {
        guildMemberOcid.value[i] = "데이터 없음";
        continue;
      }

      if (guildMemberOcid.value[i] !== "데이터 없음") {
        try {
          guildMemberMain.value[i] = await axios.get(`${API}ranking/union?date=${currentDate.value}&ocid=${guildMemberOcid.value[i].data.ocid}`, {
            headers: {
              'x-nxopen-api-key': API_KEY,
              'accept': 'application/json',
            },
          });

          for (let j = 0; j < guildMemberMain.value[i].data.ranking.length; j++) {
            if (guildMemberMain.value[i].data.ranking[j].world_name === worldname.value) {
              show.value[i] = guildMemberMain.value[i].data.ranking[j].character_name;
            }
          }
        } catch (error) {

          continue;
        }

        try {
          guildMemberMainOcid.value[i] = await axios.get(`${API}id?character_name=${show.value[i]}`, {
            headers: {
              'x-nxopen-api-key': API_KEY,
              'accept': 'application/json',
            },
          });
        }
        catch (error) {
          continue;
        }

        try {
          guildMemberMainGuild.value[i] = await axios.get(`${API}character/basic?ocid=${guildMemberMainOcid.value[i].data.ocid}&date=${currentDate.value}`, {
            headers: {
              'x-nxopen-api-key': API_KEY,
              'accept': 'application/json',
            },
          });
        }
        catch (error) {
          continue;
        }
        show3.value[i] = guildMemberMainGuild.value[i].data.character_guild_name;
      }
      else {
        guildMemberMain.value[i] = "접속기록 없음";
        show.value[i] = "접속기록 없음";
      }
    }

  } catch (error) {
    console.error("Error fetching userOcid:", error.response);
  }
}
</script>

<style>
.center-table {
  margin: 0 auto;
  /* 가운데 정렬을 위한 스타일 */
}

table {
  width: 50%;
  border-collapse: collapse;
  margin-top: 20px;
}

th,
td {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 8px;
}

th {
  background-color: #f2f2f2;
}

.no-data-row {
  background-color: #fff8e1;
  /* 옅은 노란색 */
}

.highlight-row {
  background-color: #ffe6e6;
  /* 옅은 빨간색 */
}

.highlight-col {
  background-color: #ffe6e6;
  /* 옅은 빨간색 */
}

.header-col {
  background-color: #f2f2f2;
  /* 회색 배경색 */
}
</style>