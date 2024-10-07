<script setup lang="ts">
import { computed, defineEmits, onMounted, ref } from 'vue';
import http from '@/utils/axios.js';
import { apiUserList, apiUserSessionList, apiSearch } from "@/api/chat";
import UserInfoShow from "@/components/chat/components/UserInfoShow.vue";
import { gen_show_name, type User } from "@/utils/common_utils";
import { api_img } from "@/api/base";
import { debounce } from 'lodash';

// "wxid": strUsrName, "nOrder": nOrder, "nUnReadCount": nUnReadCount, "strNickName": strNickName,
// "nStatus": nStatus, "nIsSend": nIsSend, "strContent": strContent, "nMsgLocalID": nMsgLocalID,
// "nMsgStatus": nMsgStatus, "nTime": nTime, "nMsgType": nMsgType, "nMsgSubType": nMsgSubType,
// "nickname": NickName, "remark": Remark, "account": Alias,
// "describe": describe, "headImgUrl": bigHeadImgUrl if bigHeadImgUrl else "",
// "ExtraBuf": ExtraBuf, "LabelIDList": tuple(LabelIDList)


const tableData = ref([]);
const loading = ref(false)
// 初始化请求session数据 START
const fetchData = async () => {
  try {
    loading.value = true
    tableData.value = await apiUserSessionList();
    loading.value = false
  } catch (error) {
    console.error('Error fetching data:', error);
    return [];
  }
};
onMounted(fetchData);
// END 初始化请求session数据 END

const handleInput = () => {
  search();
  console.log(1)
}

// 搜索框以及按钮 START
const search_word = ref('');
const _search = ref('')
const search = debounce(async () => {
  try {
    _search.value = search_word.value;
    // console.log(body_data);
    if (search_word.value === '') {
      tableData.value = [];
      tableDataSearch.value = [];
      tableData.value = await apiUserSessionList();
      return;
    }
    onSearch();
  } catch (error) {
    console.error('Error fetching data:', error);
    return [];
  }
}, 300)
// END 搜索框以及按钮 END


const tableDataSearch = ref([])
const contact = ref([]) // 联系人
const groupChat = ref([]) //群组
const chatRecord = ref([]) //聊天记录
// const 
// 聚合搜索
const onSearch = async () => {
  console.log(search_word.value);
  tableData.value = []
  tableDataSearch.value = []
  loading.value = true
  const ret = await apiSearch({
    query: search_word.value,
    type: "",
    page: 1,
    pagesize: 1000,
  });
  console.log(ret, "ret")
  loading.value = false
  if (ret && ret.contact) {
    contact.value = ret.contact.items || []
  }
  if (ret && ret.groupChat) {
    groupChat.value = ret.groupChat.items || []
  }
  if (ret && ret.chatRecord) {
    chatRecord.value = ret.chatRecord.items || []
  }
}

const listLength = computed(() => {
  return contact.value.length + groupChat.value.length + chatRecord.value.length
})

// 文字高亮
// const highlightedText = computed(() => (keyword: string) => {
//   const urlRegex = new RegExp(`(${search_word.value})`, 'g');
//   if (keyword) {
//     return keyword.replace(urlRegex, `<span style="color: blue;">$1</span>`);
//   } else {
//     return keyword;
//   }
// })

// 处理user数据 传递给父组件 START
const emits = defineEmits(['wxid']);

const handleCurrentChange = (val: User | undefined) => {
  // 触发自定义事件，并传递数据
  if (val !== undefined) {
    // 处理user数据
    // 判断val是否有wxid
    if (val.wxid !== undefined) {
      console.log('wxid:', val.wxid);
      emits('wxid', val.wxid);
    }
  }
}

// END 处理user数据 传递给父组件 END


// 定义一个阈值2
const numMore = 2
const contactShowAll = ref(false)
const newContact = computed(() => {
  return contactShowAll.value ? contact.value : contact.value.slice(0, numMore);
});
const toggleContactShowAll = () => {
  contactShowAll.value = !contactShowAll.value;
}

const groupChatShowAll = ref(false)
const newGroupChat = computed(() => {
  return groupChatShowAll.value ? groupChat.value : groupChat.value.slice(0, numMore);
});
const togglegroupChatShowAll = () => {
  groupChatShowAll.value = !groupChatShowAll.value;
}

const chatRecordShowAll = ref(false)
const newChatRecord = computed(() => {
  return chatRecordShowAll.value ? chatRecord.value : chatRecord.value.slice(0, numMore);
});
const togglechatRecordShowAll = () => {
  chatRecordShowAll.value = !chatRecordShowAll.value;
} 
</script>

<template>
  <div class="left-content">
    <!-- 搜索框以及按钮   -->
    <div style="padding: 10px 10px;">
      <el-input placeholder="请输入关键字" v-model="search_word" @keyup.enter="search" style="width: 100%;"
        @input="handleInput"></el-input>
      <!-- <el-button type="primary" @click="search" style="width: 50px;">搜索</el-button> -->
    </div>
    <!--  这是联系人的list    -->
    <div v-if="search_word.length == 0 && (_search == search_word)">
      <el-table :data="tableData" stripe style="width: 100%" max-height="100%" height="100%" highlight-current-row
        loading="lazy" @current-change="handleCurrentChange" v-loading="loading">
        <el-table-column width="57">
          <template v-slot="{ row }">
            <el-avatar :size="33" :src="api_img(row.headImgUrl)" v-if="row.headImgUrl !== ''"></el-avatar>
            <el-avatar :size="33" v-else>群</el-avatar>
          </template>
        </el-table-column>
        <el-table-column width="190">
          <template v-slot="{ row }">
            <el-tooltip class="item" effect="light" placement="right">
              <div slot="content" class="tips">
                <span>{{ gen_show_name(row) }}</span> <br>
                <span v-if="row.nTime" style="color: #909399; font-size: 12px;">{{ row.nTime }}</span>
              </div>
              <template #content>
                <user-info-show :userinfo="row" :show_all="false" style="max-width: 600px"></user-info-show>
              </template>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- 输入值搜索更多 -->
    <div v-if="search_word.length > 0 && (_search == search_word)" class="message-content"
      v-loading="search_word.length && loading">
      <!-- 联系人 -->
      <div class="group" v-if="contact.length > 0">
        <div class="category-title">联系人</div>
        <div v-for="(item, index) in newContact" class="message">
          <div class="logo">
            <el-avatar :size="33" :src="api_img(item.thumbnail)" v-if="item.thumbnail !== ''"></el-avatar>
            <el-avatar :size="33" v-else>群</el-avatar>
          </div>
          <div class="content">
            <div class="title" v-html="item.title"></div>
          </div>
        </div>
        <div  v-if="contact.length>numMore"  class="more" @click="toggleContactShowAll">{{ contactShowAll ? '收起' : `显示全部(${contact.length})` }}</div>
      </div>
      <!-- 群聊 -->
      <div class="group" v-if="groupChat.length > 0">
        <div class="category-title">群聊</div>
        <div v-for="(item, index) in newGroupChat" class="message">
          <div class="logo">
            <el-avatar :size="33">群</el-avatar>
          </div>
          <div class="content">
            <div class="title" v-html="item.title"></div>
            <div class="bottom" v-html="`包含:${item.note}`"></div>
          </div>
        </div>
        <div  v-if="groupChat.length>numMore"  class="more" @click="togglegroupChatShowAll">{{ groupChatShowAll ? '收起' : `显示全部(${groupChat.length})` }}</div>
      </div>
      <!-- 聊天记录 -->
      <div class="group" v-if="chatRecord.length > 0">
        <div class="category-title">聊天记录</div>
        <div v-for="(item, index) in newChatRecord" class="message">
          <div class="logo">
            <el-avatar :size="33" :src="api_img(item.thumbnail)" v-if="item.thumbnail !== ''"></el-avatar>
            <el-avatar :size="33" v-else>群</el-avatar>
          </div>
          <div class="content">
            <div class="title" v-html="item.title"></div>
            <div class="bottom"  v-html="item.note"></div>
          </div>
        </div>
        <div v-if="chatRecord.length>numMore" class="more" @click="togglechatRecordShowAll">{{ chatRecordShowAll ? '收起' : `显示全部(${chatRecord.length})` }}</div>
      </div>
      <div v-if="search_word.length > 0 && listLength == 0" class="empty">
        No Data
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
/* 允许提示内容换行 */
.el-tooltip__popper .popper__content {
  white-space: pre-line;
  /* 允许换行 */
}



.message-content {
  gap: 5px;
  display: flex;
  flex-direction: column;
  background: #ededed;

  .group {
    background: #ffffff;
  }

  .category-title {
    font-size: 12px;
    padding: 8px;
    color: #747474;
    border-bottom: 1px solid #ebeef5;

  }
}

.message {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  padding: 8px 12px;
  border-bottom: 1px solid #ebeef5;

  &:nth-child(even) {
    background: #fafafa;
  }

  &:hover {
    background: #f5f7fa;
    cursor: pointer;
  }

  .logo {
    margin-right: 24px;
    display: flex;
    align-items: center;
  }

  .content {
    display: flex;
    flex-direction: column;

    .title {
      box-sizing: border-box;
      line-height: 24px;
      overflow: hidden;
      overflow-wrap: break-word;
      text-overflow: ellipsis;
      white-space: normal;
      font-size: 14px;
      color: #606266;
      margin-bottom: 5px;
    }

    .bottom {
      color: rgb(144, 147, 153);
      font-size: 12px;
    }
  }
}

.left-content {
  width: 250px;
}

.empty {
  min-height: 200px;
  font-size: 14px;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #ffffff;
  color: #909399;
  border-top: 1px solid #ebeef5;
  border-bottom: 1px solid #ebeef5;
}

.more{
  font-size: 12px;
  padding: 5px 10px;
  border-bottom: 1px solid #ebeef5;
  cursor: pointer;
  color: #576b95;
}
</style>