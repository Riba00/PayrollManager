<template>
  <BasicLayout>
    <div class="payrolls">
      <h1>My Payrolls</h1>
      <UploadPayroll :getPayrolls="getPayrolls"/>
      <PayrollList :payrolls="payrolls" :getPayrolls="getPayrolls"/>
    </div>
  </BasicLayout>
</template>

<script>
import BasicLayout from "../layouts/BasicLayout";
import UploadPayroll from "../components/Payrolls/UploadPayroll";
import PayrollList from "../components/Payrolls/PayrollList";
import { auth, db } from "../utils/firebase";
import { ref, onMounted } from "vue";

export default {
  name: "Payrolls",
  components: {
    BasicLayout,
    UploadPayroll,
    PayrollList,
  },

  setup() {
    let payrolls = ref(null);

    onMounted(() => {
      getPayrolls();
    });
    const getPayrolls = async () => {
      const response = await db
        .collection(auth.currentUser.uid)
        .orderBy("date", "desc")
        .get();
      const result = [];
      response.forEach((doc) => {
        const data = doc.data();
        data.id = doc.id;
        result.push(data);
      });
      payrolls.value = result;
    };

    return {
      payrolls,
      getPayrolls
    }
  },
};
</script>

<style lang="scss" scoped>
.payrolls {
  margin: 50px 0;
  text-align: center;
}
</style>
