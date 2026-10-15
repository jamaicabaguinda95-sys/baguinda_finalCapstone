<script setup>
import { ref, computed, watch } from 'vue'

const students = ref(
  JSON.parse(localStorage.getItem('students') || '[]')
)

const showForm = ref(false)
const editingId = ref(null)
const search = ref('')

const emptyForm = () => ({
  studentId: '',
  name: '',
  course: '',
  year: '',
  email: ''
})

const form = ref(emptyForm())

watch(students, (value) => {
  localStorage.setItem('students', JSON.stringify(value))
}, { deep: true })

const filteredStudents = computed(() => {
  const term = search.value.toLowerCase().trim()
  return students.value.filter(student =>
    [
      student.studentId,
      student.name,
      student.course,
      student.year,
      student.email
    ].some(value => String(value).toLowerCase().includes(term))
  )
})

function openAddForm() {
  editingId.value = null
  form.value = emptyForm()
  showForm.value = true
}

function editStudent(student) {
  editingId.value = student.id
  form.value = { ...student }
  showForm.value = true
}

function saveStudent() {
  const data = { ...form.value }

  if (editingId.value !== null) {
    const index = students.value.findIndex(
      student => student.id === editingId.value
    )
    if (index !== -1) students.value[index] = {
      ...data,
      id: editingId.value
    }
  } else {
    students.value.push({
      ...data,
      id: Date.now()
    })
  }

  showForm.value = false
  form.value = emptyForm()
  editingId.value = null
}

function deleteStudent(id) {
  if (confirm('Are you sure you want to delete this student?')) {
    students.value = students.value.filter(student => student.id !== id)
  }
}
</script>

<template>
  <main class="container">
    <header class="page-header">
      <h1>Student Management System</h1>
      <p>Manage student records in one place.</p>
    </header>

    <section class="toolbar">
      <input
        v-model="search"
        type="search"
        placeholder="Search students..."
        aria-label="Search students"
      />
      <button class="primary" @click="openAddForm">
        + Add Student
      </button>
    </section>

    <section class="panel">
      <div class="panel-heading">
        <h2>Students</h2>
        <span class="count">{{ filteredStudents.length }} students</span>
      </div>

      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>ID</th>
              <th>Student ID</th>
              <th>Name</th>
              <th>Course</th>
              <th>Year</th>
              <th>Email</th>
              <th>Actions</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="(student, index) in filteredStudents" :key="student.id">
              <td>{{ index + 1 }}</td>
              <td>{{ student.studentId }}</td>
              <td>{{ student.name }}</td>
              <td>{{ student.course }}</td>
              <td>{{ student.year }}</td>
              <td>{{ student.email }}</td>
              <td class="actions">
                <button class="edit" @click="editStudent(student)">
                  Edit
                </button>
                <button class="delete" @click="deleteStudent(student.id)">
                  Delete
                </button>
              </td>
            </tr>

            <tr v-if="filteredStudents.length === 0">
              <td colspan="7" class="empty">
                {{ search ? 'No matching students found.' : 'No students yet. Click Add Student to begin.' }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <div v-if="showForm" class="overlay" @click.self="showForm = false">
      <section class="modal">
        <h2>{{ editingId !== null ? 'Edit Student' : 'Add Student' }}</h2>

        <form @submit.prevent="saveStudent">
          <label>
            Student ID
            <input v-model.trim="form.studentId" required />
          </label>

          <label>
            Full Name
            <input v-model.trim="form.name" required />
          </label>

          <label>
            Course
            <input v-model.trim="form.course" required />
          </label>

          <label>
            Year Level
            <select v-model="form.year" required>
              <option value="" disabled>Select year level</option>
              <option>1st Year</option>
              <option>2nd Year</option>
              <option>3rd Year</option>
              <option>4th Year</option>
              <option>5th Year</option>
            </select>
          </label>

          <label>
            Email
            <input v-model.trim="form.email" type="email" required />
          </label>

          <div class="form-actions">
            <button type="button" class="cancel" @click="showForm = false">
              Cancel
            </button>
            <button type="submit" class="primary">
              {{ editingId !== null ? 'Update Student' : 'Save Student' }}
            </button>
          </div>
        </form>
      </section>
    </div>
  </main>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, Helvetica, sans-serif;
  background: #f3f5f9;
  color: #1e293b;
}

button, input, select {
  font: inherit;
}

button {
  cursor: pointer;
  border: none;
  border-radius: 6px;
  padding: 10px 14px;
}

.container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 32px;
}

.page-header {
  text-align: center;
  margin-bottom: 28px;
}

.page-header h1 {
  margin: 0 0 8px;
  font-size: clamp(25px, 4vw, 34px);
}

.page-header p {
  color: #64748b;
  margin: 0;
}

.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  margin-bottom: 20px;
}

.toolbar input {
  max-width: 360px;
}

input, select {
  width: 100%;
  padding: 11px 12px;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  background: white;
}

input:focus, select:focus {
  outline: 2px solid #93c5fd;
  border-color: #0284c7;
}

.primary {
  background: #0284c7;
  color: white;
}

.primary:hover {
  background: #0369a1;
}

.panel {
  background: white;
  padding: 24px;
  border-radius: 12px;
  box-shadow: 0 3px 12px #0f172a12;
}

.panel-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 18px;
}

.panel-heading h2 {
  margin: 0;
  color: #0f172a;
}

.count {
  color: #64748b;
  font-size: 14px;
}

.table-wrap {
  width: 100%;
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
}

th {
  background: #0284c7;
  color: white;
  padding: 13px 12px;
  white-space: nowrap;
}

td {
  padding: 13px 12px;
  border-bottom: 1px solid #e2e8f0;
}

tbody tr:hover {
  background: #f8fafc;
}

.empty {
  text-align: center;
  padding: 28px;
  color: #64748b;
}

.actions {
  white-space: nowrap;
}

.edit {
  background: #dbeafe;
  color: #1d4ed8;
  margin-right: 6px;
}

.delete {
  background: #fee2e2;
  color: #b91c1c;
}

.overlay {
  position: fixed;
  inset: 0;
  z-index: 10;
  background: #0f172a80;
  display: grid;
  place-items: center;
  padding: 20px;
  overflow-y: auto;
}

.modal {
  width: 100%;
  max-width: 480px;
  background: white;
  border-radius: 12px;
  padding: 26px;
  box-shadow: 0 20px 50px #0002;
  margin: auto;
}

.modal h2 {
  margin-top: 0;
  margin-bottom: 22px;
}

.modal form {
  display: grid;
  gap: 16px;
}

.modal label {
  display: grid;
  gap: 7px;
  font-weight: 600;
  font-size: 14px;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 8px;
}

.cancel {
  background: #e2e8f0;
  color: #334155;
}

@media (max-width: 650px) {
  .container {
    padding: 18px 12px;
  }

  .toolbar {
    align-items: stretch;
    flex-direction: column;
  }

  .toolbar input {
    max-width: none;
  }

  .panel {
    padding: 15px;
  }
}
</style>