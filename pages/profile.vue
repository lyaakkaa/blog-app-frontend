<template>
  <div>
    <Header :toggleMenu="toggleMenu" />
    <div class="profile-container">
      <h1>User Profile</h1>

      <div v-if="user" class="user-info">
        <img
          :src="user.avatar || '/src/assets/no_pfp.svg'"
          alt="User Avatar"
          class="user-avatar"
        />
        <div class="user-details">
          <template v-if="editMode">
            <input
              type="text"
              v-model="user.person_name"
              placeholder="Name"
              class="input-field"
            />
            <input
              type="email"
              v-model="user.email"
              placeholder="Email"
              class="input-field"
            />
            <input
              type="text"
              v-model="user.login"
              placeholder="Username"
              class="input-field"
            />
            <input
              type="password"
              v-model="user.password"
              placeholder="Password"
              class="input-field"
            />
            <input
              type="number"
              v-model="user.age"
              placeholder="Age"
              class="input-field"
            />
            <input
              type="text"
              v-model="user.location"
              placeholder="Location"
              class="input-field"
            />
            <input type="file" @change="onFileChange" class="input-file" />
          </template>
          <template v-else>
            <h2>{{ user.person_name }}</h2>
            <p class="user-info-text">
              Email: {{ user.email || "Not provided" }}
            </p>
            <p class="user-info-text">
              Username: {{ user.login || "Not provided" }}
            </p>
            <p class="user-info-text">Age: {{ user.age || "Not provided" }}</p>
            <p class="user-info-text">
              Location: {{ user.location || "Not provided" }}
            </p>
            <p class="user-info-text">
              Last Activity:
              {{
                user.activity
                  ? new Date(user.activity).toLocaleString()
                  : "Not provided"
              }}
            </p>
          </template>
        </div>
      </div>

      <div class="button-group">
        <button @click="toggleEditMode" class="edit-button">
          {{ editMode ? "Cancel" : "Edit Profile" }}
        </button>
        <button v-if="editMode" @click="saveChanges" class="save-button">
          Save
        </button>
      </div>

      <div class="user-posts">
        <h2>Your Posts</h2>
        <div v-if="posts.length">
          <div v-for="post in posts" :key="post.id" class="post-item">
            <div class="post-header">
              <h3>{{ post.commentary }}</h3>
              <p class="post-date">{{ post.pub_date }}</p>
            </div>
            <p class="post-content">Topic: {{ post.topic.name }}</p>
            <p class="post-rating">Rating: {{ post.rating }} ⭐</p>
          </div>
        </div>
        <p v-else class="no-posts">No posts available.</p>
      </div>

      <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
    </div>
  </div>
</template>

<script>
import Header from "~/components/Header.vue";

export default {
  components: {
    Header,
  },
  data() {
    return {
      user: null,
      posts: [],
      editMode: false,
      newAvatar: null,
      errorMessage: "",
    };
  },
  async mounted() {
    const userId = localStorage.getItem("userId");
    if (!userId) {
      this.errorMessage = "User not found.";
      return;
    }

    try {
      const userResponse = await fetch(
        `http://127.0.0.1:8000/api/users/${userId}`
      );
      const userData = await userResponse.json();

      if (userResponse.ok) {
        this.user = userData;
      } else {
        this.errorMessage =
          userData.error || "Failed to load user information.";
      }

      const postsResponse = await fetch(
        `http://127.0.0.1:8000/api/users/${userId}/posts`
      );
      const postsData = await postsResponse.json();
      if (postsResponse.ok) {
        this.posts = postsData;
      } else {
        this.errorMessage = postsData.error || "Failed to load user posts.";
      }
    } catch (error) {
      this.errorMessage = "Error loading profile data. Please try again.";
    }
  },
  methods: {
    toggleMenu() {
      this.menuVisible = !this.menuVisible;
    },
    toggleEditMode() {
      this.editMode = !this.editMode;
    },
    onFileChange(event) {
      this.newAvatar = event.target.files[0];
    },
    async saveChanges() {
      const userId = localStorage.getItem("userId");
      const formData = new FormData();
      formData.append("person_name", this.user.person_name);
      formData.append("email", this.user.email);
      formData.append("login", this.user.login);
      formData.append("password", this.user.password);
      formData.append("age", this.user.age);
      formData.append("location", this.user.location || null);
      formData.append("activity", this.user.activity);
      if (this.newAvatar) {
        formData.append("avatar", this.newAvatar);
      }

      try {
        const response = await fetch(
          `http://127.0.0.1:8000/api/users/${userId}/`,
          {
            method: "PUT",
            body: formData,
          }
        );

        if (response.ok) {
          this.editMode = false;
          this.errorMessage = "";
          const updatedUser = await response.json();
          this.user = updatedUser;
        } else {
          const errorData = await response.json();
          this.errorMessage = errorData.error || "Failed to update profile.";
        }
      } catch (error) {
        this.errorMessage = "Error saving changes. Please try again.";
      }
    },
  },
};
</script>

<style scoped>
.profile-container {
  max-width: 500px; /* Reduced the width */
  margin: 20px auto;
  padding: 20px; /* Reduced padding */
  text-align: center;
  background-color: #ffffff;
  border-radius: 8px; /* Reduced border-radius */
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1); /* Softer shadow */
}

h1 {
  color: #333;
  font-weight: bold;
  margin-bottom: 15px; /* Reduced margin */
  font-size: 24px; /* Slightly smaller font size */
}

.user-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 15px; /* Reduced padding */
  background-color: #f9f9f9;
  border-radius: 6px; /* Reduced border-radius */
  margin-bottom: 15px;
  text-align: center;
}

.user-avatar {
  border-radius: 50%;
  width: 70px; /* Reduced avatar size */
  height: 70px; /* Reduced avatar size */
  margin-bottom: 10px; /* Adjusted spacing */
}

.user-details {
  margin: 0 auto;
  text-align: center;
}

.user-details h2 {
  font-size: 18px; /* Reduced font size */
  color: #333;
  margin: 0;
}

.user-info-text {
  color: #666;
  font-size: 12px; /* Reduced font size */
  margin: 4px 0;
}

.input-field {
  width: 100%;
  padding: 8px; /* Reduced padding */
  margin: 6px 0; /* Reduced margin */
  border: 1px solid #ddd;
  border-radius: 4px; /* Reduced border-radius */
}

.input-file {
  margin-top: 8px; /* Reduced margin */
}

.button-group {
  display: flex;
  gap: 8px; /* Reduced gap */
  justify-content: center;
  margin-bottom: 15px; /* Reduced margin */
}

.edit-button,
.save-button {
  padding: 6px 12px; /* Reduced button padding */
  border: none;
  border-radius: 4px; /* Reduced border-radius */
  font-size: 14px; /* Reduced font size */
  font-weight: bold;
  cursor: pointer;
}

.edit-button {
  background-color: #4a90e2;
  color: white;
}

.save-button {
  background-color: #27ae60;
  color: white;
}

.user-posts h2 {
  color: #333;
  font-weight: bold;
  margin-bottom: 8px; /* Reduced margin */
  font-size: 18px; /* Reduced font size */
}

.post-item {
  background-color: #f5f5f5;
  padding: 12px; /* Reduced padding */
  border-radius: 4px; /* Reduced border-radius */
  margin-bottom: 10px; /* Reduced margin */
  text-align: left;
}

.post-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.post-header h3 {
  color: #333;
  font-size: 14px; /* Reduced font size */
}

.post-date {
  font-size: 10px; /* Reduced font size */
  color: #999;
}

.post-content,
.post-rating {
  font-size: 12px; /* Reduced font size */
  color: #555;
}

.error-message {
  color: #d32f2f;
  margin-top: 0.75rem; /* Reduced margin */
}
</style>
