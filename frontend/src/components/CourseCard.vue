<template>
	<div
		v-if="course.title"
		class="si-course-card group"
	>
		<!-- Silicon Course Image -->
		<div
			class="si-course-image relative overflow-hidden"
			:class="{ 'default-image': !course.image }"
			:style="{ backgroundImage: course.image ? 'url(\'' + encodeURI(course.image) + '\')' : '' }"
		>
			<!-- Course Badges -->
			<div class="absolute top-4 left-4 flex flex-wrap gap-2">
				<span
					v-if="course.featured"
					class="si-badge si-badge-success"
				>
					{{ __('Featured') }}
				</span>
				<span
					v-if="course.tags"
					v-for="tag in course.tags?.split(', ')"
					:key="tag"
					class="si-badge si-badge-primary"
				>
					{{ tag }}
				</span>
			</div>

			<!-- Default Image Placeholder -->
			<div v-if="!course.image" class="image-placeholder">
				{{ course.title[0] }}
			</div>

			<!-- Hover Overlay -->
			<div class="absolute inset-0 bg-black bg-opacity-0 group-hover:bg-opacity-20 transition-all duration-300 flex items-center justify-center">
				<div class="opacity-0 group-hover:opacity-100 transition-opacity duration-300">
					<span class="si-btn si-btn-primary text-sm">
						{{ __('View Course') }}
					</span>
				</div>
			</div>
		</div>

		<!-- Silicon Course Content -->
		<div class="si-course-content">
			<!-- Course Meta Information -->
			<div class="si-course-meta">
				<div v-if="course.lessons" class="flex items-center text-gray-600">
					<BookOpen class="h-4 w-4 stroke-1.5 mr-1" />
					<span class="text-sm">{{ course.lessons }} {{ __('lessons') }}</span>
				</div>

				<div v-if="course.enrollments" class="flex items-center text-gray-600">
					<Users class="h-4 w-4 stroke-1.5 mr-1" />
					<span class="text-sm">{{ course.enrollments }} {{ __('students') }}</span>
				</div>

				<div v-if="course.rating" class="flex items-center text-gray-600">
					<Star class="h-4 w-4 stroke-1.5 mr-1 fill-yellow-400 text-yellow-400" />
					<span class="text-sm">{{ course.rating }}</span>
				</div>

				<div v-if="course.status != 'Approved'">
					<span
						class="si-badge"
						:class="{
							'si-badge-warning': course.status === 'Under Review',
							'si-badge-primary': course.status !== 'Under Review'
						}"
					>
						{{ course.status }}
					</span>
				</div>
			</div>

			<!-- Course Title -->
			<h3 class="si-card-title">
				{{ course.title }}
			</h3>

			<!-- Course Description -->
			<p class="si-card-description">
				{{ course.short_introduction }}
			</p>

			<!-- Progress Bar for Enrolled Users -->
			<div v-if="user && course.membership" class="mb-4">
				<ProgressBar :progress="course.membership.progress" />
				<div class="text-sm text-gray-600 mt-1">
					{{ Math.ceil(course.membership.progress) }}% {{ __('completed') }}
				</div>
			</div>

			<!-- Course Footer -->
			<div class="si-card-footer">
				<!-- Course Instructors -->
				<div class="si-course-instructor">
					<div class="flex items-center -space-x-2">
						<UserAvatar
							v-for="instructor in course.instructors"
							:key="instructor.name"
							:user="instructor"
							class="w-6 h-6 border-2 border-white"
						/>
					</div>
					<CourseInstructors :instructors="course.instructors" />
				</div>

				<!-- Course Price and Certification -->
				<div class="flex items-center gap-3">
					<div v-if="course.paid_course" class="font-semibold text-lg text-gray-900">
						{{ course.price }}
					</div>
					<div
						v-if="course.paid_certificate || course.enable_certification"
						class="si-badge si-badge-primary"
					>
						{{ __('Certificate') }}
					</div>
				</div>
			</div>
		</div>
	</div>
</template>
<script setup>
import { BookOpen, Users, Star } from 'lucide-vue-next'
import UserAvatar from '@/components/UserAvatar.vue'
import { sessionStore } from '@/stores/session'
import { Badge, Tooltip } from 'frappe-ui'
import CourseInstructors from '@/components/CourseInstructors.vue'
import ProgressBar from '@/components/ProgressBar.vue'

const { user } = sessionStore()

const props = defineProps({
	course: {
		type: Object,
		default: null,
	},
})
</script>
<style>
/* Silicon Course Image Styles */
.si-course-image {
	height: 200px;
	width: 100%;
	background-size: cover;
	background-position: center;
	background-repeat: no-repeat;
	border-radius: var(--si-radius-lg) var(--si-radius-lg) 0 0;
}

.default-image {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	background: linear-gradient(135deg, var(--si-primary-100) 0%, var(--si-primary-200) 100%);
	color: var(--si-primary-700);
}

.image-placeholder {
	font-size: 4rem;
	font-weight: 700;
	text-transform: uppercase;
}

/* Course Description Truncation */
.si-card-description {
	display: -webkit-box;
	-webkit-line-clamp: 2;
	line-clamp: 2;
	-webkit-box-orient: vertical;
	text-overflow: ellipsis;
	overflow: hidden;
	line-height: 1.5;
}

/* Avatar Group Styling */
.avatar-group {
	display: inline-flex;
	align-items: center;
}

.avatar-group .avatar {
	transition: margin 0.1s ease-in-out;
}

.avatar-group.overlap .avatar + .avatar {
	margin-left: -8px;
}

/* Responsive adjustments */
@media (max-width: 768px) {
	.si-course-image {
		height: 160px;
	}
	
	.image-placeholder {
		font-size: 3rem;
	}
}
</style>
