<template>
  <div v-if="!errorCode">
    <!-- 文件和集数列表 -->
    <DriveSelector :drive-list="driveList" :selected-drive="selectedDrive" @change-drive="changeDrive" />
    <FileListLoading v-if="loading"></FileListLoading>
    <FileListMain :la-data="laData" :file-list="fileList" :selected-file="selectedFile" v-if="!loading"
      class="sm:mb-4" />
  </div>
  <div v-else>
    <AnimeBasicCard class="py-8 sm:mb-4">
      <n-result status="info" title="出现错误" :description="'错误代码: ' + errorCode"></n-result>
    </AnimeBasicCard>
  </div>
</template>

<script>
import { LavaAnimeAPI } from '../../common/api';
import AnimeBasicCard from '../../components/Anime/Cards/AnimeBasicCard.vue';

export default {
  inject: ["changePlayingFile"],
  props: {
    laID: Number,
    laData: Object,
    selectedFile: Object
  },
  data() {
    return {
      loading: true,
      driveList: {},
      selectedDrive: "",
      fileList: [],
      errorCode: null
    };
  },
  mounted() {
    if (this.laID) {
      console.log("mounted 触发文件刷新");
      this.reborn();
    }
  },
  watch: {
    laID() {
      console.log("watch laID 触发文件刷新");
      this.reborn();
    }
  },
  methods: {
    // 重置函数，界面第一次挂载和参数切换时均会调用
    async reborn() {
      // 重置参数
      this.loading = true;
      this.driveList = {};
      this.selectedDrive = "";
      this.fileList = [];
      // 获取资源节点相关
      let driveListResult = await this.getDriveList();
      this.driveList = driveListResult;
      this.selectedDrive = driveListResult.default || driveListResult.list[0].id;
      // 获取相应资源节点下的资源相关
      let fileListResult = await this.getFileList(this.laID, this.selectedDrive);
      this.fileList = fileListResult;
      // 结束
      this.loading = false;
    },
    async getDriveList() {
      try {
        let driveList = (await LavaAnimeAPI.get("/v2/drive/all")).data;
        console.log(`Got DriveList`, driveList);
        return driveList.data;
      }
      catch (error) {
        console.error("获取存储节点列表时发生错误: ", error);
        this.errorCode = error.response?.status;
        return {};
      }
    },
    async getFileList(laID, drive) {
      try {
        let result = (await LavaAnimeAPI.get("/v2/anime/file", { params: { id: laID, drive: drive } })).data;
        console.log(`Got FileList of la${this.laID}:`, result);
        return result.data;
      }
      catch (error) {
        console.error("获取视频文件列表时发生错误: ", error);
        this.errorCode = error.response?.status;
        return [];
      }
    },
    async changeDrive(newDrive) {
      if (this.selectedDrive == newDrive)
        return;
      this.selectedDrive = newDrive;
      this.loading = true;
      this.changePlayingFile({});
      let fileListResult = await this.getFileList(this.laID, this.selectedDrive);
      this.fileList = fileListResult;
      this.loading = false;
    }
  },
  components: { AnimeBasicCard }
}
</script>