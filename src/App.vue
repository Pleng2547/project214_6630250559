<template>
  <div class="container">
    <div class="profile-section">
      <img src="../src/assets/IMG_0686.jpg" alt="Profile" class="profile-img">
      <div class="profile-info">
        <h2>{{ profile.Name }}</h2>
        <p>Nice too meet you! 🎉</p>
      </div>
    </div>

    <div class="info-section">
      <h2 class="info-title">ข้อมูลส่วนตัว</h2>
      <div class="profile-content">
        <div class="additional-image">
          <img src="../src/assets/IMG_3914.jpg" alt="Profile" class="profile1-img">
        </div>
        <div class="grid-container">
          <p><strong>รหัสนิสิต:</strong> {{ profile.studentid }}</p>
          <p><strong>ชื่อ:</strong> {{ profile.Name }}</p>
          <p><strong>รหัสสาขา:</strong> {{ profile.majorID }}</p>
          <p><strong>สาขา:</strong> {{ profile.majorName }}</p>
          <p><strong>โรงเรียนเดิม:</strong> {{ profile.oldschool }}</p>
        </div>
      </div>
      <button class="edit-btn" @click="editStudent">แก้ไขข้อมูล</button>
    </div>

    <div class="table-container">
      <h2>ตารางวิชา</h2>
      <button class="add-btn" @click="openAddPopup">+ เพิ่มรายวิชา</button>
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>รหัสวิชา</th>
            <th>ชื่อวิชา</th>
            <th>หน่วยกิต</th>
            <th>เกรด</th>
            <th>จัดการ</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(subject, index) in subjects" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ subject.id }}</td>
            <td>{{ subject.name }}</td>
            <td>{{ subject.credits }}</td>
            <td>{{ subject.grade }}</td>
            <td>
              <button class="edit-subject action-btn" @click="editSubject(subject)">แก้ไข</button>
              <button class="delete-subject action-btn" @click="removeSubject(subject.id)">ลบ</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Popup สำหรับแก้ไขข้อมูลส่วนตัว -->
    <div v-if="editProfileMode" class="popup">
      <div class="popup-content">
        <h3>แก้ไขข้อมูลส่วนตัว</h3>
        <form @submit.prevent="saveProfile">
          <label for="studentid">รหัสนิสิต:</label>
          <input type="text" id="studentid" v-model="profile.studentid" required>

          <label for="name">ชื่อ:</label>
          <input type="text" id="name" v-model="profile.Name" required>

          <label for="majorID">รหัสสาขา:</label>
          <input type="text" id="majorID" v-model="profile.majorID" required>

          <label for="majorName">สาขา:</label>
          <input type="text" id="majorName" v-model="profile.majorName" required>

          <label for="oldschool">โรงเรียนเดิม:</label>
          <input type="text" id="oldschool" v-model="profile.oldschool" required>

          <button type="submit" class="save-btn">บันทึก</button>
          <button type="button" class="cancel-btn" @click="closeProfilePopup">ยกเลิก</button>
        </form>
      </div>
    </div>

    <!-- Popup สำหรับเพิ่ม/แก้ไขวิชา -->
    <div v-if="editSubjectMode" class="popup">
      <div class="popup-content">
        <h3>{{ isAdding ? "เพิ่มรายวิชา" : "แก้ไขรายวิชา" }}</h3>
        <form @submit.prevent="saveSubject">
          <label for="subject-id">รหัสวิชา:</label>
          <input type="text" id="subject-id" v-model="editedSubject.id" required>

          <label for="subject-name">ชื่อวิชา:</label>
          <input type="text" id="subject-name" v-model="editedSubject.name" required>

          <label for="subject-credits">หน่วยกิต:</label>
          <input type="number" id="subject-credits" v-model="editedSubject.credits" required>

          <label for="subject-grade">เกรด:</label>
          <input type="text" id="subject-grade" v-model="editedSubject.grade" required>

          <button type="submit" class="save-btn">บันทึก</button>
          <button type="button" class="cancel-btn" @click="closePopup">ยกเลิก</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      profile: {
        studentid: "",
        Name: "",
        majorID: "",
        majorName: "",
        oldschool: "",
      },
      subjects: [],
      editProfileMode: false,  // ใช้สำหรับเปิด/ปิด Popup ข้อมูลส่วนตัว
      editSubjectMode: false,
      editedSubject: { id: "", name: "", credits: 0, grade: "" },
      isAdding: false,
    };
  },
  methods: {
    async saveProfile() {
      try {
        const response = await fetch("http://localhost:3000/user", {
          method: "PUT",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(this.profile),
        });

        if (response.ok) {
          this.editProfileMode = false; // ปิด Popup หลังบันทึก
        } else {
          alert("บันทึกข้อมูลไม่สำเร็จ");
        }
      } catch (error) {
        console.error("Error saving profile:", error);
      }
    },
    editStudent() {
      this.editProfileMode = true;  // เปิด Popup สำหรับแก้ไขข้อมูลส่วนตัว
    },
    closeProfilePopup() {
      this.editProfileMode = false;  // ปิด Popup ข้อมูลส่วนตัว
    },
    async fetchSubjects() {
      try {
        const response = await fetch("http://localhost:3000/subjects");
        this.subjects = await response.json();
      } catch (error) {
        console.error("Error fetching subjects:", error);
      }
    },
    openAddPopup() {
      this.editedSubject = { id: "", name: "", credits: 0, grade: "" };
      this.isAdding = true;
      this.editSubjectMode = true;
    },
    async removeSubject(id) {
      try {
        await fetch(`http://localhost:3000/subjects/${id}`, { method: "DELETE" });
        this.fetchSubjects();
      } catch (error) {
        console.error("Error deleting subject:", error);
      }
    },
    editSubject(subject) {
      this.editedSubject = { ...subject };
      this.id_present = subject.id;
      this.isAdding = false;
      this.editSubjectMode = true;
    },
    async saveSubject() {
      try {
        if (this.isAdding) {
          await fetch("http://localhost:3000/subjects", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(this.editedSubject),
          });
        } else {
          await fetch(`http://localhost:3000/subjects/${this.id_present}`, {
            method: "DELETE",
            headers: { "Content-Type": "application/json" },
          });

          await fetch("http://localhost:3000/subjects", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(this.editedSubject)
          });
        }
        this.editSubjectMode = false;
        this.fetchSubjects();
      } catch (error) {
        console.error("Error saving subject:", error);
      }
    },
    closePopup() {
      this.editSubjectMode = false;  // ปิด Popup วิชา
    },
  },
  async mounted() {
    try {
      const response = await fetch("http://localhost:3000/user");
      const data = await response.json();
      if (data) this.profile = data;
      this.fetchSubjects();
    } catch (error) {
      console.error("Error fetching profile:", error);
    }
  },
};
</script>

<style scoped>
/* General container styles */
.container {
  font-family: Arial, sans-serif;
  margin: 20px;
  padding: 20px;
  background-color: #e0f7fa;
  border-radius: 10px;
}

/* Profile section */
.profile-section {
  display: flex;
  align-items: center;
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.profile-info h2 {
  font-size: 24px;
  margin-bottom: 10px;
}

.profile-info p {
  font-size: 16px;
}
.profile-img {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  margin-right: 20px; 
}

/* Info section */
.info-section {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.profile-content{
  display: flex;
}

.additional-image {
  margin-right: 20px; /* ระยะห่างระหว่างรูปภาพและข้อมูล */
}
.grid-container {
  display: flex;
  flex-direction: column; /* ให้ข้อมูลใน grid เป็นคอลัมน์ */
  margin-left: 30px;
}
.profile1-img {
  width: 250px;
  height: 250px;
}

button {
  cursor: pointer;
  padding: 8px 16px;
  border: none;
  border-radius: 5px;
  margin-top: 15px;
}

.edit-btn {
  background-color: #03a9f4; /* ฟ้าสีเข้ม */
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 5px;
  margin-top: 20px;
  cursor: pointer;
}

.add-btn {
  background-color: #03a9f4; /* ฟ้าสด */
  color: white;
}

.edit-subject,
.delete-subject {
  margin-right: 10px;
  padding: 8px 12px;
}

.action-btn {
  border-radius: 5px;
  padding: 6px 10px;
  cursor: pointer;
  font-size: 14px;
}

.edit-subject {
  background-color: #81d4fa; /* ฟ้าสีอ่อน */
}

.delete-subject {
  background-color: #f44336; /* สีแดงสด */
  color: white;
}

/* Table styles */
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

table th,
table td {
  padding: 10px;
  text-align: left;
  border: 1px solid #ddd;
}

table th {
  background-color: #b3e5fc; /* ฟ้าสดอ่อน */
  font-weight: bold;
}

/* Popup styles */
.popup {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.popup-content {
  background: white;
  padding: 20px;
  border-radius: 10px;
  width: 400px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
}

.popup-content h3 {
  font-size: 22px;
  margin-bottom: 20px;
  color: #0288d1; /* ฟ้าสีเข้ม */
}

.popup-content form {
  display: flex;
  flex-direction: column;
}

.popup-content input {
  margin-bottom: 15px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
}

.save-btn,
.cancel-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  margin-top: 10px;
  font-size: 16px;
}

.save-btn {
  background-color: #0288d1; /* ฟ้าสีเข้ม */
  color: white;
}

.cancel-btn {
  background-color: #dc3545; /* สีแดงสด */
  color: white;
}

/* Additional styles for responsiveness */
@media screen and (max-width: 768px) {
  .profile-section {
    flex-direction: column;
    align-items: flex-start;
  }

  .profile-img {
    margin-bottom: 20px;
  }

  .grid-container {
    grid-template-columns: 1fr;
  }

  .popup-content {
    width: 90%;
  }
}

</style>