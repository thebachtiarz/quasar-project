<template>
  <div id="biodata-management">
    <p class="text-bold text-center">--- Biodata Management ---</p>
    <div class="form-group">
      <label for="setting-biodata-name">Full Name</label>
      <input
        type="text"
        class="form-control"
        id="setting-biodata-name"
        v-model="fullNameBiodata"
      />
    </div>
    <div class="form-group">
      <form
        id="setting-biodata-image"
        enctype="multipart/form-data"
        @submit.prevent="uploadImageProfile"
      >
        <label for="setting-biodata-image-import">Image Profile</label>
        <div class="custom-file">
          <input
            type="file"
            class="custom-file-input"
            name="_image"
            id="setting-biodata-image-import"
            accept="image/x-png, image/jpeg"
            @change="autoUploadImageOnChange"
          />
          <label
            class="custom-file-label"
            for="setting-biodata-image-import"
          >{{imageUploadOrigin}}</label>
        </div>
        <input
          type="text"
          id="setting-biodata-image-src"
          v-model="imageUploadForm"
          hidden
          readonly
        />
        <input
          type="submit"
          class="btn btn-primary"
          id="setting-biodata-image-submit"
          style="display: none;"
        />
      </form>
      <p
        class="text-muted mt-1"
        style="font-size: 12px"
      >Image rule: photo dimensions 4x4, format [jpg, jpeg, png], max size 2 MB.</p>
      <img
        v-if="this.imageUploadResult"
        class="profile-user-img img-fluid img-circle"
        :src="`${this.$AppHelper.apiEndpoint()}${this.imageUploadResult}`"
        alt="user-profile-image"
      />
    </div>
    <button
      class="btn btn-primary float-right text-bold"
      @click="biodataUpdatePost"
    >
      <i class="fas fa-user-alt"></i>&ensp;Update Biodata
    </button>
  </div>
</template>

<script>
import { mapActions } from 'vuex'
import Swal from 'sweetalert2'
export default {
  name: 'ManagementBiodata',
  methods: {
    ...mapActions('AuthStore', ['getBiodata']),
    autoUploadImageOnChange () {
      this.$('#setting-biodata-image-submit').click()
    },
    uploadImageProfile () {
      this.$axios.getCookies().then(() => {
        const formImage = new FormData(
          document.getElementById('setting-biodata-image')
        )
        this.$axios
          .postNewImageUserProfile(formImage)
          .then(res => {
            if (res.data.status === 'success') {
              this.imageUploadOrigin = res.data.response_data[0].origin_name
              this.imageUploadForm = res.data.response_data[0].img_name
              this.imageUploadResult = res.data.response_data[0].img_name
            } else {
              this.imageUploadOrigin = ''
              this.imageUploadForm = ''
              this.imageUploadResult = ''
              this.$Notify.notifyError(`${res.data.message._image[0]}`)
            }
          })
          .catch(err => this.catchError('toast', err))
      })
    },
    biodataUpdatePost () {
      Swal.fire({
        title: 'Update your biodata?',
        icon: 'question',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        cancelButtonText: 'Cancel!',
        confirmButtonText: 'Yes, Update Now!'
      }).then(async result => {
        if (result.value) {
          if (this.fullNameBiodata) {
            this.$axios.getCookies().then(() => {
              this.$axios
                .patchUserBiodata(
                  this.appTimeNow,
                  this.fullNameBiodata,
                  this.imageUploadForm
                )
                .then(async res => {
                  if (res.data.status === 'success') {
                    await Swal.fire(
                      'Success',
                      `${res.data.message}`,
                      'success'
                    )
                    this.imageUploadOrigin = ''
                    this.imageUploadForm = ''
                    this.imageUploadResult = ''
                    this.getBiodata()
                  } else {
                    await Swal.fire(
                      'Opps!!',
                      `${this.responseFailed(res.data.message)}`,
                      'warning'
                    )
                  }
                })
                .catch(err => this.catchError('swal', err))
            })
          }
          if (!this.fullNameBiodata) {
            Swal.fire('Waitt!!', 'You cannot leave your name blank', 'warning')
          }
        }
      })
    },
    responseFailed (data) {
      var errorMsg = ''
      if (data.name) {
        data.name.forEach(msg => {
          errorMsg += errorMsg ? `, ${msg}` : msg
        })
      }
      if (data.old_pass) {
        data.old_pass.forEach(msg => {
          errorMsg += errorMsg ? `, ${msg}` : msg
        })
      }
      if (data.new_pass) {
        data.new_pass.forEach(msg => {
          errorMsg += errorMsg ? `, ${msg}` : msg
        })
      }
      if (errorMsg) return errorMsg
      else return data
    },
    setFieldMessage (goto, status, message) {
      this.$(`${goto}`).css(
        'color',
        `${status === 'success' ? '#119822' : '#C91E1E'}`
      )
      this.$(`${goto}`).html(message)
    },
    catchError (alertType, err) {
      alertType === 'swal'
        ? Swal.fire('Sorry', 'Opps!, something went wrong', 'error')
        : this.$Notify.notifyError('Opps!, something went wrong')
      console.log(err)
    }
  },
  data () {
    return {
      appTimeNow: new Date().getTime(),
      fullNameBiodata: this.$store.state.AuthStore.thisBiodata.name,
      imageUploadOrigin: '',
      imageUploadForm: '',
      imageUploadResult: ''
    }
  }
}
</script>