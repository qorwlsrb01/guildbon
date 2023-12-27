<template>
  <div>
    홈
    <form @submit.prevent="submitForm1">
      <label for="guildname">길드 이름:</label>
      <input v-model="guildname" type="text" id="guildname" name="guildname">
      <label for="worldname">서버명:</label>
      <input v-model="worldname" type="text" id="worldname" name="worldname">
      <button type="submit">확인</button>
    </form>
    <div v-if="ocidDataGuild">
      <!-- <div v-if="showflag === true"> -->
      <div v-for="(member, index) in guildBasicData.guild_member" :key="index" class="member-container">
        {{ index + 1 }} {{ guildBasicData.guild_member[index] }}
        || 본캐 : {{ show[index] }}
        || 본캐 길드 : {{ show3[index] }}
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';

const API = "https://open.api.nexon.com/maplestory/v1/";
const API_KEY = "test_9961b9606a2eeae06255293a5e7e010a9fcd6d0df07c1fe33729cb8fc25318aa2e07f03ae396b640fd77e08c3509a738";

const ocidData = ref('');
const characterBasicData = ref('');
const username = ref('');
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
const show2 = ref(['']);
const show3 = ref(['']);


async function submitForm1() {
  console.log("Submitted guildname:", guildname.value);
  await getGuildOcid();
  await getGuildBasic();
}

async function submitForm() {
  console.log("Submitted username:", username.value);
  await getUserOcid();
  await getCharacterBasic();
}

async function getGuildOcid() {
  try {
    const guildOcid = await axios.get(`${API}guild/id?guild_name=${guildname.value}&world_name=${worldname.value}`, {
      headers: {
        'x-nxopen-api-key': API_KEY,
        'accept': 'application/json',
      },
    });
    console.log(guildOcid.data);
    ocidDataGuild.value = guildOcid.data;
    console.log(ocidDataGuild.value.oguild_id);
  } catch (error) {
    console.error("Error fetching userOcid:", error.response.data);
  }
}

async function getGuildBasic() {
  try {
    const guildBasic = await axios.get(`${API}guild/basic?oguild_id=${ocidDataGuild.value.oguild_id}&date=2023-12-22`, {
      headers: {
        'x-nxopen-api-key': API_KEY,
        'accept': 'application/json',
      },
    });
    console.log(guildBasic.data);
    guildBasicData.value = guildBasic.data;
    console.log(guildBasicData.value.guild_member);
    for (let i = 0; i < guildBasicData.value.guild_member_count; i++) {
      // console.log(i);
      guildMember.value[i] = guildBasicData.value.guild_member[i];
      try {
        guildMemberOcid.value[i] = await axios.get(`${API}id?character_name=${guildMember.value[i]}`, {
          headers: {
            'x-nxopen-api-key': API_KEY,
            'accept': 'application/json',
          },
        });
      } catch (error) {
        console.error("Error fetching userOcid:", error.response.data);
        guildMemberOcid.value[i] = "데이터 없음";
        continue;
      }
      console.log(guildMember.value[i]);
      console.log(guildMemberOcid.value[i].data.ocid);

      if (guildMemberOcid.value[i] !== "데이터 없음") {
        try {
          guildMemberMain.value[i] = await axios.get(`${API}ranking/union?date=2023-12-22&ocid=${guildMemberOcid.value[i].data.ocid}`, {
            headers: {
              'x-nxopen-api-key': API_KEY,
              'accept': 'application/json',
            },
          });
          console.log(guildMemberMain.value[i].data.ranking.length)

          for (let j = 0; j < guildMemberMain.value[i].data.ranking.length; j++) {
            if (guildMemberMain.value[i].data.ranking[j].world_name === worldname.value) {
              show.value[i] = guildMemberMain.value[i].data.ranking[j].character_name;
            }
          }
          // console.log(guildMemberMain.value[i].data.ranking[0].character_name);
        } catch (error) {

          continue;
        }

        console.log("본캐 ocid 찾기")
        try {
          guildMemberMainOcid.value[i] = await axios.get(`${API}id?character_name=${show.value[i]}`, {
            headers: {
              'x-nxopen-api-key': API_KEY,
              'accept': 'application/json',
            },
          });
          console.log(guildMemberMainOcid.value[i].data.ocid);
        }
        catch (error) {
          continue;
        }

        console.log("본캐길드 찾기")
        try {
          guildMemberMainGuild.value[i] = await axios.get(`${API}character/basic?ocid=${guildMemberMainOcid.value[i].data.ocid}&date=2023-12-22`, {
            headers: {
              'x-nxopen-api-key': API_KEY,
              'accept': 'application/json',
            },
          });
        }
        catch (error) {
          continue;
        }
        show3.value[i]=guildMemberMainGuild.value[i].data.character_guild_name;
      }
      else {
        guildMemberMain.value[i] = "접속기록 없음";
        show.value[i] = "접속기록 없음";
        console.log("error")
      }
    }

    console.log(guildMember.value);
    console.log(guildMemberOcid.value);
    console.log(show.value);
    console.log(guildBasicData.value.guild_member_count)
  } catch (error) {
    console.error("Error fetching userOcid:", error.response);
  }
}
</script>

<style ></style>