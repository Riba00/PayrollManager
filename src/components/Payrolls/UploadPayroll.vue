<template>
  <div class="upload-payroll">
    <button @click="showCloseForm" class="ui button primary">
      New Payroll
    </button>
  </div>

  <form
    class="ui form error upload-payroll__form"
    :class="{ open: showForm }"
    @submit.prevent="handleSubmit"
  >
    <div class="field">
      <label for="file" class="ui icon button">
        <i class="file icon" />
        Select Payroll
        <span v-if="file">({{ file.name }})</span>
      </label>
      <input type="file" id="file" style="display: none" @change="uploadFile" />
    </div>

    <div class="field">
      <div class="ui calendar">
        <div class="ui input left icon">
          <input type="date" @change="onChangeDate" />
        </div>
      </div>
    </div>

    <div class="ui error message" v-if="errorMessage">
      <p class="header">{{ errorMessage }}</p>
    </div>

    <button class="ui button positive" :class="{ loading: isLoading }">
      Upload Payroll
    </button>
  </form>
</template>

<script>
import { ref } from "vue";
import { auth, storage, db } from "../../utils/firebase";
import { v4 as uuidv4 } from "uuid";
import Swal from "sweetalert2";

export default {
  name: "UploadPayroll",
  props: {
    getPayrolls: Function
  },
  setup(props) {
    let showForm = ref(false);
    let file = ref(null);
    let date = ref(null);
    let isLoading = ref(false);
    let errorMessage = ref(null);

    const showCloseForm = () => {
      showForm.value = !showForm.value;
    };

    const uploadFile = (e) => {
      errorMessage.value = null;
      const fileTemp = e.target.files[0];

      if (fileTemp?.type === "application/pdf") {
        file.value = fileTemp;
      } else {
        errorMessage.value = "Invalid File Format. Please upload a PDF file";
      }
    };

    const onChangeDate = (e) => {
      date.value = e.target.value;
    };

    const handleSubmit = async () => {
      if (file.value && date.value) {
        isLoading.value = true;

        try {
          const fileName = uuidv4();
          await storage
            .ref(auth.currentUser.uid)
            .child(`${fileName}.pdf`)
            .put(file.value);

          const payrollUrl = await storage.ref(`${auth.currentUser.uid}/${fileName}.pdf`).getDownloadURL();

          await db.collection(auth.currentUser.uid).add({
            payrollUrl,
            date: new Date(date.value),
            dateString: date.value,
          });
          
          props.getPayrolls();

          Swal.fire({
            position: "center",
            icon: "success",
            title: "Payroll uploaded",
            showConfirmButton: false,
            timer: 1500,
          });

          showForm.value = false;
          file.value = null;
          date.value = null;
          errorMessage.value = null;
          document.getElementById("file").value = "";
        } catch (error) {
          console.log(error);
          errorMessage.value = "An error occurred while uploading the file";
        }
      } else {
        errorMessage.value = "Please select a file and a date";
      }
      isLoading.value = false;
    };

    return {
      showCloseForm,
      uploadFile,
      onChangeDate,
      handleSubmit,
      showForm,
      file,
      date,
      isLoading,
      errorMessage,
    };
  },
};
</script>

<style lang="scss" scoped>
.upload-payroll {
  > .ui.button.primary {
    margin-bottom: 30px;
  }

  &__form {
    display: flex;
    align-items: center;
    flex-direction: column;
    height: 0;
    overflow: hidden;
    transition: height 0.3s ease;

    &.open {
      height: 250px;
    }
  }
}
</style>
