<template>
  <div class="container py-5">
    <div class="row justify-content-center">
      <div class="col-12 col-md-8 col-lg-6">

        <div class="card shadow-lg border-0 rounded-4">

          <!-- Header -->
          <div class="card-header text-center text-white rounded-top-4"
            style="background: linear-gradient(135deg, #0d6efd, #0dcaf0);">
            <h4 class="mb-1 fw-bold">📋 ระบบเบิกอุปกรณ์</h4>
            <small>กรอกข้อมูลเพื่อส่งเข้า n8n Webhook</small>
          </div>

          <!-- Body -->
          <div class="card-body p-4">

            <!-- Alert -->
            <div v-if="status.message"
              :class="`alert alert-${status.type} alert-dismissible fade show`">
              {{ status.message }}
              <button type="button" class="btn-close"
                @click="status.message = ''"></button>
            </div>

            <!-- Form -->
            <form @submit.prevent="submitForm">

              <!-- ชื่อ -->
              <div class="mb-3">
                <label class="form-label">ชื่อ-นามสกุล *</label>
                <input type="text" class="form-control"
                  v-model="data.fullname" required>
              </div>

              <!-- แผนก -->
              <div class="mb-3">
                <label class="form-label">แผนก *</label>
                <select class="form-select" v-model="data.department" required>
                  <option value="" disabled>-- เลือกแผนก --</option>
                  <option value="บุคคล">บุคคล</option>
                  <option value="บัญชี">บัญชี</option>
                  <option value="ไอที">ไอที</option>
                  <option value="การตลาด">การตลาด</option>
                  <option value="ประชาสัมพันธ์">ประชาสัมพันธ์</option>
                </select>
              </div>

              <!-- อุปกรณ์ -->
              <div class="mb-3">
                <label class="form-label">รายการอุปกรณ์ *</label>
                <select class="form-select" v-model="data.item" required>
                  <option value="" disabled>-- เลือกอุปกรณ์ --</option>
                  <option value="กรรไกร">กรรไกร</option>
                  <option value="เทปกาว">เทปกาว</option>
                  <option value="ปากกา">ปากกา</option>
                  <option value="แฟ้มเอกสาร">แฟ้มเอกสาร</option>
                  <option value="คัตเตอร์">คัตเตอร์</option>
                </select>
              </div>

              <!-- จำนวน -->
              <div class="mb-3">
                <label class="form-label">จำนวน *</label>
                <input type="number" class="form-control"
                  v-model="data.qty" min="1" required>
              </div>

              <!-- ปุ่ม -->
              <button class="btn btn-primary w-100 fw-bold"
                :disabled="loading">

                <span v-if="loading"
                  class="spinner-border spinner-border-sm me-2"></span>

                {{ loading ? 'กำลังส่งข้อมูล...' : 'บันทึกข้อมูล' }}
              </button>

            </form>

          </div>

        </div>

      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from "vue";

const data = reactive({
  fullname: "",
  department: "",
  item: "",
  qty: ""
});

const loading = ref(false);

const status = reactive({
  message: "",
  type: ""
});

const submitForm = async () => {
  loading.value = true;
  status.message = "";

  try {
    const response = await fetch("http://localhost:5678/webhook/equipment", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        fullname: data.fullname,
        department: data.department,
        item: data.item,
        qty: data.qty,
        date: new Date().toISOString().split("T")[0]
      })
    });

    if (!response.ok) throw new Error("Error");

    const result = await response.json();
    console.log(result);

    status.message = "✅ บันทึกข้อมูลสำเร็จ!";
    status.type = "success";

    // reset
    data.fullname = "";
    data.department = "";
    data.item = "";
    data.qty = "";

  } catch (error) {
    console.error(error);
    status.message = "❌ เกิดข้อผิดพลาด กรุณาลองใหม่";
    status.type = "danger";
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
.card {
  transition: 0.3s;
}
.card:hover {
  transform: translateY(-5px);
}
.form-control:focus {
  box-shadow: 0 0 0 0.2rem rgba(13, 110, 253, 0.2);
  border-color: #0d6efd;
}
</style>