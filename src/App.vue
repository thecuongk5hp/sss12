<template>
  <div class="container mt-4">
    <div class="d-flex justify-content-between align-items-center mb-3">
      <h2 class="mb-0">Quản lý mượn trả sách</h2>
      <div>
        <select
          v-model="statusFilter"
          class="form-select me-2 d-inline-block"
          style="width: auto"
        >
          <option value="all">Lọc theo trạng thái</option>
          <option value="returned">Đã trả</option>
          <option value="not-returned">Chưa trả</option>
        </select>
        <button class="btn btn-primary" @click="showAddAndEditForm">
          Thêm thông tin
        </button>
      </div>
    </div>

    <!-- Bảng hiển thị thông tin mượn sách -->
    <div class="table-responsive">
      <table class="table table-bordered table-hover">
        <thead class="table-light">
          <tr>
            <th>STT</th>
            <th>Tên sách</th>
            <th>Sinh viên mượn</th>
            <th>Ngày mượn</th>
            <th>Ngày trả</th>
            <th>Trạng thái</th>
            <th>Chức năng</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(book, index) in filteredBooks" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ book.title }}</td>
            <td>{{ book.student }}</td>
            <td>{{ formatDate(book.borrowDate) }}</td>
            <td>{{ formatDate(book.returnDate) }}</td>
            <td>
              <button
                :class="[
                  'btn badge',
                  book.returned ? 'bg-success' : 'bg-danger',
                ]"
                @click="toggleBookStatus(index)"
              >
                {{ book.returned ? "Đã trả" : "Chưa trả" }}
              </button>
            </td>
            <td>
              <button
                class="btn btn-warning btn-sm me-1"
                @click="showEditForm(index)"
              >
                Sửa
              </button>
              <button
                class="btn btn-danger btn-sm"
                @click="confirmDelete(index)"
              >
                Xóa
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Form thêm/sửa sách -->
    <div v-if="showForm" class="modal-overlay">
      <div class="modal-content">
        <h5>
          {{
            isEditing
              ? "Cập nhật thông tin mượn sách"
              : "Thêm thông tin mượn sách"
          }}
        </h5>
        <form @submit.prevent="submitForm">
          <div class="mb-3">
            <label for="title" class="form-label">Tên sách</label>
            <input
              v-model="currentBook.title"
              type="text"
              id="title"
              class="form-control"
              required
            />
          </div>
          <div class="mb-3">
            <label for="student" class="form-label">Tên người mượn</label>
            <input
              v-model="currentBook.student"
              type="text"
              id="student"
              class="form-control"
              required
            />
          </div>
          <div class="mb-3">
            <label for="borrowDate" class="form-label">Ngày mượn</label>
            <input
              v-model="currentBook.borrowDate"
              type="date"
              id="borrowDate"
              class="form-control"
              required
            />
          </div>
          <div class="mb-3">
            <label for="returnDate" class="form-label">Ngày trả</label>
            <input
              v-model="currentBook.returnDate"
              type="date"
              id="returnDate"
              class="form-control"
              required
            />
          </div>
          <div class="d-flex justify-content-end">
            <button type="submit" class="btn btn-primary">
              {{ isEditing ? "Cập nhật" : "Thêm" }}
            </button>
            <button
              type="button"
              class="btn btn-secondary ms-2"
              @click="closeForm"
            >
              Đóng
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal xác nhận xóa -->
    <div v-if="showDeleteModal" class="modal-overlay">
      <div class="modal-content">
        <h5>Xác nhận xóa</h5>
        <p>Bạn có chắc chắn muốn xóa thông tin này không?</p>
        <div class="d-flex justify-content-end">
          <button class="btn btn-danger" @click="deleteBook">Xóa</button>
          <button
            class="btn btn-secondary ms-2"
            @click="showDeleteModal = false"
          >
            Hủy
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

// Lấy dữ liệu từ localStorage hoặc khởi tạo danh sách sách rỗng
const books = ref(JSON.parse(localStorage.getItem("books")) || []);
const statusFilter = ref("all");
const showForm = ref(false);
const showDeleteModal = ref(false);
const isEditing = ref(false);
const currentBook = ref({
  title: "",
  student: "",
  borrowDate: "",
  returnDate: "",
  returned: false,
});
let bookToDelete = ref(null);
let bookToEdit = ref(null);

// Hàm định dạng ngày
const formatDate = (date) => {
  return new Date(date).toLocaleDateString("vi-VN");
};

// Danh sách sách lọc theo trạng thái
const filteredBooks = computed(() => {
  if (statusFilter.value === "all") return books.value;
  return books.value.filter(
    (book) =>
      (statusFilter.value === "returned" && book.returned) ||
      (statusFilter.value === "not-returned" && !book.returned)
  );
});

// Hiển thị form thêm sách
const showAddAndEditForm = () => {
  isEditing.value = false;
  currentBook.value = {
    title: "",
    student: "",
    borrowDate: "",
    returnDate: "",
    returned: false,
  };
  showForm.value = true;
};

// Hiển thị form sửa sách
const showEditForm = (index) => {
  isEditing.value = true;
  bookToEdit.value = index;
  currentBook.value = { ...books.value[index] };
  showForm.value = true;
};

// Đóng form
const closeForm = () => {
  showForm.value = false;
  currentBook.value = {
    title: "",
    student: "",
    borrowDate: "",
    returnDate: "",
    returned: false,
  };
};

// Xử lý submit form (thêm hoặc sửa)
const submitForm = () => {
  const today = new Date().toISOString().split("T")[0];
  if (currentBook.value.borrowDate > currentBook.value.returnDate) {
    alert("Ngày trả không được nhỏ hơn ngày mượn.");
    return;
  }

  if (isEditing.value) {
    books.value[bookToEdit.value] = { ...currentBook.value };
  } else {
    books.value.push({ ...currentBook.value });
  }

  saveBooks();
  closeForm();
};

// Hiển thị modal xác nhận xóa
const confirmDelete = (index) => {
  bookToDelete.value = index;
  showDeleteModal.value = true;
};

// Hàm xóa sách
const deleteBook = () => {
  books.value.splice(bookToDelete.value, 1);
  saveBooks();
  showDeleteModal.value = false;
};

// Hàm thay đổi trạng thái sách
const toggleBookStatus = (index) => {
  books.value[index].returned = !books.value[index].returned;
  saveBooks();
};

// Hàm lưu sách vào localStorage
const saveBooks = () => {
  localStorage.setItem("books", JSON.stringify(books.value));
};
</script>

<style>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: #fff;
  padding: 20px;
  border-radius: 4px;
  width: 500px;
}
</style>
