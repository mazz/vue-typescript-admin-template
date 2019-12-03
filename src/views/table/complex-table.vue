<template>
  <div class="app-container">
    <el-dropdown trigger="click" @command="handleSetChannel">
      <div>
        <el-col :span="4" class="text-center">
          <router-link class="pan-btn green-btn" to="/table/complex-table">
            Select Channel
          </router-link>
        </el-col>
      </div>
      <el-dropdown-menu slot="dropdown">
        <el-dropdown-item v-for="item in channels" :key="item.channel_uuid" :label="item.basename+'('+item.channel_id+')'" :value="item.channel_uuid" :command="item.channel_uuid">
          {{
            item.basename }}
        </el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>

    <div class="filter-container">
      <!-- <el-select v-model="listQuery.channel" placeholder="Channel" clearable class="filter-item" style="width: 130px"> -->
      <!-- <el-option v-for="item in channels" :key="item.uuid" :label="item.basename+'('+item.uuid+')'" :value="item.uuid" /> -->
      <!-- </el-select> -->
      <!-- <el-input v-model="listQuery.title" placeholder="Localized Name" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" /> -->
      <!-- <el-select v-model="listQuery.type" placeholder="Media Category" clearable class="filter-item" style="width: 130px"> -->
      <!-- <el-option v-for="item in mediaCategoryOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key" /> -->
      <!-- </el-select> -->
      <!-- <el-select v-model="listQuery.sort" style="width: 140px" class="filter-item" @change="handleFilter"> -->
      <!-- <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key" /> -->
      <!-- </el-select> -->
      <!-- <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter"> -->
      <!-- Search -->
      <!-- </el-button> -->
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        Add
      </el-button>
      <!-- <el-button v-waves :loading="downloadLoading" class="filter-item" type="primary" icon="el-icon-download" @click="handleDownload"> -->
      <!-- Export -->
      <!-- </el-button> -->
    </div>

    <el-table
      :key="tableKey"
      v-loading="playlistsLoading"
      :data="playlists"
      border
      fit
      highlight-current-row
      style="width: 100%;"
      @sort-change="sortChange"
    >
      <el-table-column label="Ordinal" prop="id" sortable="custom" align="center" width="80" :class-name="getSortClass('ordinal')">
        <template slot-scope="{row}">
          <span>{{ row.ordinal }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Updated At" width="150px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.updated_at | parseTime('{y}-{m}-{d} {h}:{i}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Base Name" min-width="200px">
        <template slot-scope="{row}">
          <span class="link-type" @click="handleUpdate(row)">{{ row.basename }}</span>
          <el-tag>{{ row.media_category | typeFilter }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="Hash ID" width="110px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.hash_id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Banner Path" min-width="150px">
        <template slot-scope="{row}">
          <span class="link-type" @click="handleUpdate(row)">{{ row.banner_path }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Small Thumbnail Path" min-width="150px">
        <template slot-scope="{row}">
          <span class="link-type" @click="handleUpdate(row)">{{ row.small_thumbnail_path }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Med Thumbnail Path" min-width="150px">
        <template slot-scope="{row}">
          <span class="link-type" @click="handleUpdate(row)">{{ row.med_thumbnail_path }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Large Thumbnail Path" min-width="150px">
        <template slot-scope="{row}">
          <span class="link-type" @click="handleUpdate(row)">{{ row.large_thumbnail_path }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Actions" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            Edit
          </el-button>
          <!-- <el-button v-if="row.status!='deleted'" size="mini" type="danger" @click="handleModifyStatus(row,'deleted')"> -->
          <!-- Delete -->
          <!-- </el-button> -->
        </template>
      </el-table-column>
    </el-table>

    <!-- <pagination v-show="totalPlaylists>0" :total="totalPlaylists" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getPlaylists" /> -->

    <!-- Main ADD/EDIT Playlist Dialog -->
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="120px" style="width: 600px; margin-left:30px;">
        <el-form-item label="Channel" prop="channel">
          <el-select v-model="temp.channel_id" placeholder="Please select">
            <el-option v-for="item in channels" :key="item.channel_id" :label="item.basename+'('+item.channel_id+')'" :value="item.channel_id" />
          </el-select>
        </el-form-item>
        <el-form-item label="Ordinal" prop="ordinal">
          <el-input v-model="temp.ordinal" />
        </el-form-item>
        <el-form-item label="Media Category" prop="media_category">
          <el-select v-model="temp.media_category" class="filter-item" placeholder="Please select">
            <el-option v-for="item in mediaCategoryOptions" :key="item.key" :label="item.display_name" :value="item.key" />
          </el-select>
        </el-form-item>
        <!-- <el-form-item label="Date" prop="updated_at">
          <el-date-picker v-model="temp.updated_at" type="datetime" placeholder="Please pick a date" />
        </el-form-item> -->
        <el-form-item label="Basename" prop="basename">
          <el-input v-model="temp.basename" />
        </el-form-item>

        <!-- <el-form-item label="Localized Titles" prop="addlocalizedtitles">
        <el-button type="primary" @click="addLocalizationDialogVisible = true">
        + Add Localized Titles
        </el-button>
        </el-form-item> -->

        <!-- <el-form-item label="Localized Name" prop="localizedname">
          <el-input v-model="temp.localizedname" />
        </el-form-item>
        <el-form-item label="Language ID" prop="language_id">
          <el-input v-model="temp.language_id" />
        </el-form-item> -->
        <el-form-item label="Banner Path" prop="banner_path">
          <el-input v-model="temp.banner_path" />
        </el-form-item>
        <el-form-item label="Small Thumbnail Path" prop="small_thumbnail_path">
          <el-input v-model="temp.small_thumbnail_path" />
        </el-form-item>
        <el-form-item label="Medium Thumbnail Path" prop="med_thumbnail_path">
          <el-input v-model="temp.med_thumbnail_path" />
        </el-form-item>
        <el-form-item label="Large Thumbnail Path" prop="large_thumbnail_path">
          <el-input v-model="temp.large_thumbnail_path" />
        </el-form-item>


        <el-form-item label="Localizations" prop="localizations">
        <el-button type="primary" @click="handleAddLocalizationDialogCreate">
          Add
        </el-button>
        </el-form-item>

      <el-table v-loading="playlistsLoading" :data="addLocalizationTitleData" border fit highlight-current-row style="width: 100%">
        <el-table-column align="center" label="Localization">
          <template slot-scope="{row}">
            <span>{{ row.language_id }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="Localized Title">
          <template slot-scope="{row}">
            <span>{{ row.localizedname }}</span>
          </template>
        </el-table-column>
<!--
        <el-table-column align="center" label="Actions">
          <template slot-scope="{row}">
            <el-button
              type="primary"
              size="small"
              @click="confirmLocalizedRowDelete(row)"
            >
              Delete
            </el-button>
          </template>
        </el-table-column>
-->
      </el-table>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          Cancel
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          <!-- <el-button @click="dialogStatus==='create'?createData():updateData()"> -->
          Confirm
        </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">Confirm</el-button>
      </span>
    </el-dialog>

    <el-dialog :visible.sync="addLocalizationDialogVisible" title="Add/Edit Localized Playlist Title">
      <el-form ref="addLocalizationForm" 
      label-position="left" 
      label-width="120px" 
      style="width: 400px; margin-left:70px;"
      >

        <!-- localized lang popover -->
        <el-form-item label="Language ID" prop="language_identifier">
          <el-select ref="select" v-model="selected_language_identifier" placeholder="en">
            <el-option v-for="item in addableLanguages" :key="item.language_identifier" :label="item.language_identifier" :value="item.language_identifier" />
          </el-select>
        </el-form-item>

        <!-- localized title field -->
        <el-form-item label="Localized Name" prop="pushLocalizedName">
          <el-input v-model="pushLocalizedName" />
        </el-form-item>

        <!-- add localization button -->
        <el-form-item>
          <el-button type="normal" @click="handleAddLocalizationTitle">
            + Add Localized Title
          </el-button>
        </el-form-item>
      </el-form>

      <!-- <el-table :data="addLocalizationTitleData">
        <el-table-column property="localization" label="Localization" width="200" />
        <el-table-column property="localizedTitle" label="Localized Title" />
      </el-table> -->

      <el-table v-loading="playlistsLoading" :data="addLocalizationTitleData" border fit highlight-current-row style="width: 100%">
        <el-table-column align="center" label="Language ID" width="200">
          <template slot-scope="{row}">
            <span>{{ row.language_id }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="Localized Name">
          <template slot-scope="{row}">
            <span>{{ row.localizedname }}</span>
          </template>
        </el-table-column>

        <el-table-column align="center" label="Actions" width="120">
          <template slot-scope="{row}">
            <el-button
              type="primary"
              size="small"
              @click="confirmLocalizedRowDelete(row)"
            >
              Delete
            </el-button>
          </template>
        </el-table-column>
      </el-table>

    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'
// import { fetchList, fetchPv, createArticle, updateArticle } from '@/api/article'
// import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

const mediaCategoryOptions = [
  { key: 'bible', display_name: 'Bible' },
  { key: 'gospel', display_name: 'Gospel' },
  { key: 'livestream', display_name: 'Livestream' },
  { key: 'motivation', display_name: 'Motivation' },
  { key: 'movie', display_name: 'Movie' },
  { key: 'music', display_name: 'Music' },
  { key: 'podcast', display_name: 'Podcast' },
  { key: 'preaching', display_name: 'Preaching' },
  { key: 'testimony', display_name: 'Testimony' },
  { key: 'tutorial', display_name: 'Tutorial' },
  { key: 'conference', display_name: 'Conference' },
  { key: 'audiobook', display_name: 'Audiobook' }
]

// arr to obj, such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = mediaCategoryOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'ComplexTable',
  // components: { Pagination },
  // directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type]
    }
  },
  data() {
    return {
      tableKey: 0,
      playlists: null,
      totalPlaylists: 0,
      playlistsLoading: true,
      addLocalizationTitleData: null,
      playlistDetailsLoading: true,
      supportedLanguages: null,
      supportedLanguagesLoading: true,
      addableLanguages: null,
      listQuery: {
        page: 1,
        limit: 20,
        importance: undefined,
        title: undefined,
        type: undefined,
        sort: '+ordinal'
      },
      channels: [],
      mediaCategoryOptions,
      sortOptions: [{ label: 'Ordinal Ascending', key: '+ordinal' }, { label: 'Ordinal Descending', key: '-ordinal' }],
      temp: {
        channel_id: null,
        ordinal: null,
        updated_at: new Date(),
        basename: null,
        media_category: null,
        banner_path: null,
        small_thumbnail_path: null,
        med_thumbnail_path: null,
        large_thumbnail_path: null
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        // channel: [{ required: true, message: 'channel is required', trigger: 'blur' }],
        ordinal: [{ required: true, message: 'ordinal is required', trigger: 'blur' }],
        media_category: [{ required: true, message: 'media category is required', trigger: 'blur' }],
        basename: [{ required: true, message: 'basename is required', trigger: 'blur' }]
      },
      downloadLoading: false,
      addLocalizationDialogVisible: false,
      selected_language_identifier: '--',
      pushLocalizedName: ''
    }
  },
  created() {
    this.getChannels()
    this.getLanguages()
  },
  methods: {
    isNumeric: function(n) {
      return !isNaN(parseFloat(n)) && isFinite(n)
    },
    getChannels() {
      // show playlists from the bible channel as default
      var bibleChannelUuid = ''

      axios.get('http://localhost:4000/api/v1.3/orgs/64c1fa34-ebe9-425b-ae58-4815d933b01c/channels?language-id=en&offset=1&limit=50'
        // { params: { type: 'all', }, }
      )
        .then((res) => {
          console.log('Success Response', res.data)
          res.data.result.forEach((channel) => {
            this.channels.push({ channel_uuid: channel.uuid, basename: channel.basename, hash_id: channel.hash_id, channel_id: channel.channel_id, updated_at: channel.updated_at })
            console.log('this.channels', this.channels)
            console.log('channel.basename', channel.basename)
            if (channel.basename == 'Bible') {
              console.log('found Bible')
              bibleChannelUuid = channel.uuid
              this.getPlaylists(bibleChannelUuid)
            }
          })
        })
        .catch((err) => {
          console.log('Error', err)
        })
      console.log('bibleChannelUuid', bibleChannelUuid)
      // this.getPlaylists()
    },
    getPlaylists(channel_uuid) {
      console.log(`getPlaylists: ${channel_uuid}`)
      if (channel_uuid != null) {
        this.playlistsLoading = true
        return axios.get(`http://localhost:4000/api/v1.3/channels/${channel_uuid}/playlists?language-id=en&offset=1&limit=1000`)
          .then(response => {
            console.log(response)
            this.playlistsLoading = false
            this.playlists = response.data.result
            this.totalPlaylists = response.data.total_entries
          })
      }
    },
    getPlaylistDetails(playlist_uuid) {
      console.log(`getPlaylistDetails: ${playlist_uuid}`)

      // editing existing playlist
      if (playlist_uuid != null) {
        this.playlistDetailsLoading = true
        return axios.get(`http://localhost:4000/api/v1.3/playlist/${playlist_uuid}/details?offset=1&limit=50`)
          .then(response => {
            console.log(response)
            this.playlistDetailsLoading = false
            console.log(`getPlaylistDetails result: ${response.data.result}`)
            console.log(`getPlaylistDetails response.data.result[0].playlist_titles: ${response.data.result[0].playlist_titles}`)
            this.addLocalizationTitleData = response.data.result[0].playlist_titles

            // extract the language identifiers from the playlist details
            var language_identifiers = response.data.result[0].playlist_titles.map(function(title) {
              return title.language_id;
            });
            console.log(`language_identifiers: ${language_identifiers}`)

            // extract the language identifiers from all the currently supported languages
            var supported_identifiers = this.supportedLanguages.map(function(languages) {
              return languages.language_identifier;
            });
            console.log(`supported_identifiers: ${supported_identifiers}`)
            
            // languages that can be added to this playlist
            var addable_identifiers = null
            addable_identifiers = supported_identifiers.filter(n => !language_identifiers.includes(n));
            console.log(`addable_identifiers: ${addable_identifiers}`)

            // generate js objects as a data model
            var addable_languages = addable_identifiers.map(function(language) {
              return { language_identifier: language };
            });
            this.addableLanguages = addable_languages
          })
      }
    },
    getLanguages() {
      console.log(`getLanguages`)
      // if (channel_uuid != null) {
        this.supportedLanguagesLoading = true
        return axios.get(`http://localhost:4000/api/v1.3/languages/supported?offset=1&limit=50`)
          .then(response => {
            console.log(response)
            this.supportedLanguagesLoading = false
            this.supportedLanguages = response.data.result
            console.log(`this.supportedLanguages: ${this.supportedLanguages}`)
          })
      // }
    },
    handleSetChannel(channel_uuid) {
      console.log(`handleSetChannel: ${channel_uuid}`)
      // this.$ELEMENT.size = size
      // this.$store.dispatch('app/setSize', size)
      // this.refreshView()
      if (channel_uuid != null) {
        this.getPlaylists(channel_uuid)
        this.$message({
          message: 'Switch Channel Success',
          type: 'success'
        })
      }
    },
    handleAddLocalizationDialogCreate() {
      this.getPlaylistDetails(this.temp.uuid) // playlist uuid
      console.log(`handleAddLocalizationDialogCreate: ${this.addLocalizationDialogVisible}`)
      this.addLocalizationDialogVisible = true
      console.log(`handleAddLocalizationDialogCreate: ${this.addLocalizationDialogVisible}`)
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getPlaylists()
    },
    handleModifyStatus(row, status) {
      this.$message({
        message: '操作Success',
        type: 'success'
      })
      row.status = status
    },
    sortChange(data) {
      const { prop, order } = data
      if (prop === 'ordinal') {
        this.sortByOrdinal(order)
      }
    },
    sortByOrdinal(order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+ordinal'
      } else {
        this.listQuery.sort = '-ordinal'
      }
      this.handleFilter()
    },
    resetRowModel() {
      this.temp = {
        channel_id: null,
        ordinal: null,
        updated_at: new Date(),
        basename: null,
        localizedname: null,
        language_id: null,
        media_category: null,
        banner_path: null,
        small_thumbnail_path: null,
        med_thumbnail_path: null,
        large_thumbnail_path: null
      }

      // for the add/edit localization titles dialog
      this.addLocalizationTitleData = null
      this.pushLocalizedName = ''
    },
    handleCreate() {
      this.resetRowModel()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      console.log('createData')
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          console.log(`valid, refs dataForm: ${this.$refs['dataForm']}`)
          console.log(`valid, temp.ordinal: ${this.temp.ordinal}`)
          console.log(`valid, temp.media_category: ${this.temp.media_category}`)
          console.log(`valid, temp.updated_at: ${this.temp.updated_at}`)
          console.log(`valid, temp.basename: ${this.temp.basename}`)
          console.log(`valid, temp.localizedname: ${this.temp.localizedname}`)
          console.log(`valid, temp.language_id: ${this.temp.language_id}`)
          console.log(`valid, temp.channel_id: ${this.temp.channel_id}`)
          console.log(`valid, temp.banner_path: ${this.temp.banner_path}`)
          console.log(`valid, temp.small_thumbnail_path: ${this.temp.small_thumbnail_path}`)
          console.log(`valid, temp.med_thumbnail_path: ${this.temp.med_thumbnail_path}`)
          console.log(`valid, temp.large_thumbnail_path: ${this.temp.large_thumbnail_path}`)

          var ordinal = Number(this.temp.ordinal)
          var language_id = this.temp.language_id
          console.log(`language_id: ${language_id}`)

          var localized_titles = []
          for(let i = 0; i< this.addLocalizationTitleData.length; i++) {

            const localization = this.addLocalizationTitleData[i].language_id
            const localizedname = this.addLocalizationTitleData[i].localizedname
            localized_titles.push({[language_id]: localizedname}) 
          }

          console.log(`localized_titles: ${localized_titles}`)

          const body = {
            ordinal: ordinal,
            media_category: this.temp.media_category,
            updated_at: this.temp.updated_at,
            basename: this.temp.basename,
            localized_titles: localized_titles,
            channel_id: this.temp.channel_id,
            banner_path: this.temp.banner_path,
            small_thumbnail_path: this.temp.small_thumbnail_path,
            med_thumbnail_path: this.temp.med_thumbnail_path,
            large_thumbnail_path: this.temp.large_thumbnail_path
          }

          axios.post('http://localhost:4000/api/v1.3/playlists/add', body)
          // .then(function (response) {
            .then((response) => {
              this.$notify({
                title: 'Success',
                message: 'Content submitted successfully',
                type: 'success',
                duration: 2000
              })
              this.dialogFormVisible = false

              //
              // just POSTed/created a new playlist so reset globals
              //

              this.channels = []
              this.resetRowModel()

              // getChannels will fetch all channels + all playlists for
              // the default channel(Bible)
              this.getChannels()
              this.getLanguages()

              console.log(`response: ${response}`)
            })
          // .catch(function (error) {
            .catch((error) => {
              this.$notify({
                title: 'Error',
                message: 'Please check the values you attempted to submit',
                type: 'error',
                duration: 2000
              })
              console.log(`error: ${error}`)
            })

          // }

          // createArticle(this.temp).then(() => {
          //   this.list.unshift(this.temp)
          //   this.$notify({
          //     title: 'Success',
          //     message: 'Created Successfully',
          //     type: 'success',
          //     duration: 2000
          //   })
          // })
        }
      })
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.temp.updated_at = new Date(this.temp.updated_at)

      console.log(`handleUpdate this.temp.uuid: ${this.temp.uuid}`) // playlist uuid

      // reset list of localizations. need to be refetched via playlist_titles
      this.addLocalizationTitleData = null

      // reset selected language identifier popup
      this.selected_language_identifier = '--'

      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          tempData.updated_at = +new Date(tempData.updated_at) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
          updateArticle(tempData).then(() => {
            for (const v of this.playlists) {
              if (v.id === this.temp.id) {
                const index = this.playlists.indexOf(v)
                this.playlists.splice(index, 1, this.temp)
                break
              }
            }
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Update Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleAddLocalizationTitle() {
      this.addLocalizationTitleData.push({ language_id: this.language_identifier, localizedname: this.pushLocalizedName })
      console.log('this.addLocalizationTitleData', this.addLocalizationTitleData)
    },
    confirmLocalizedRowDelete(row) {
      console.log('confirmLocalizedRowDelete', row)
      this.$message({
        message: 'The title has been deleted',
        type: 'success'
      })
      const index = this.addLocalizationTitleData.indexOf(row)
      this.addLocalizationTitleData.splice(index, 1)
    },
    handleDelete(row) {
      this.$notify({
        title: 'Success',
        message: 'Delete Successfully',
        type: 'success',
        duration: 2000
      })
      const index = this.playlists.indexOf(row)
      this.playlists.splice(index, 1)
    },
    handleFetchPv(pv) {
      fetchPv(pv).then(response => {
        this.pvData = response.data.pvData
        this.dialogPvVisible = true
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => {
        if (j === 'updated_at') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    },
    getSortClass: function(key) {
      const sort = this.listQuery.sort
      return sort === `+${key}`
        ? 'ascending'
        : sort === `-${key}`
          ? 'descending'
          : ''
    }
  }
}
</script>
