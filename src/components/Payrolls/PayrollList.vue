<template>
  <div class="payroll-list">
    <p class="payroll-list__not-found" v-if="payrolls?.length === 0">
      No payrolls found, please upload a payroll.
    </p>
    <div
      class="payroll-list__payroll"
      v-for="payroll in payrolls"
      :key="payroll.id"
    >
      <p>{{ formatDate(payroll.dateString) }}</p>
      <div class="action">
        <a :href="payroll.payrollUrl" target="_blank" class="ui button positive"
          >Download</a
        >
        <button class="ui button red" @click="deletePayroll(payroll.id)">
          Delete
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import { auth, db } from "../../utils/firebase";
import Swal from "sweetalert2";

export default {
  name: "PayrollList",
  props: {
    payrolls: Array,
    getPayrolls: Function,
  },
  setup(props) {
    const formatDate = (date) => {
      return moment(date).format("MMMM YYYY");
    };

    const deletePayroll = async (id) => {
      const result = await Swal.fire({
        title: "Are you sure?",
        text: "You won’t be able to revert this!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, delete it!",
      });

      if (result.isConfirmed) {
        try {
          await db.collection(auth.currentUser.uid).doc(id).delete();
          Swal.fire("Deleted!", "Your payroll has been deleted.", "success");
          props.getPayrolls();
        } catch (error) {
          Swal.fire(
            "Error!",
            "There was an error deleting the payroll.",
            "error"
          );
        }
      }
    };

    return {
      formatDate,
      deletePayroll,
    };
  },
};
</script>

<style lang="scss" scoped>
.payroll-list {
  &__not-found {
    text-align: center;
    font-size: 20px;
  }

  &__payroll {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin: 15px 0;

    p {
      margin: 0;
      text-transform: uppercase;
      font-weight: bold;
    }
  }
}
</style>
