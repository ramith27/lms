<template>
	<div v-if="course.data">
		<!-- Silicon Navigation Header -->
		<header class="si-nav">
			<div class="si-nav-content">
				<Breadcrumbs class="h-7" :items="breadcrumbs" />
			</div>
		</header>		<!-- Silicon Course Hero Section - Ultra Compact Layout -->
		<section class="relative bg-gradient-to-br from-blue-50 to-indigo-100 py-6 lg:py-8">
			<div class="max-w-7xl mx-auto px-4 lg:px-6">
				<div class="grid lg:grid-cols-3 gap-4 lg:gap-6 items-start">
					<!-- Course Information -->
					<div class="lg:col-span-2">
						<div class="si-animate-fadeInUp">
							<h1 class="text-xl lg:text-2xl xl:text-3xl font-bold text-gray-900 mb-2 lg:mb-3 leading-tight">
								{{ course.data.title }}
							</h1>
							
							<p class="text-sm lg:text-base text-gray-600 mb-3 lg:mb-4 leading-relaxed">
								{{ course.data.short_introduction }}
							</p>

							<!-- Course Stats - Ultra Compact Layout -->
							<div class="flex flex-wrap items-center gap-2 lg:gap-3 mb-3 lg:mb-4">
								<div v-if="parseInt(course.data.rating) > 0" class="flex items-center">
									<Star class="h-3.5 w-3.5 text-yellow-400 fill-yellow-400 mr-1" />
									<span class="font-semibold text-gray-900 mr-1 text-xs">{{ course.data.rating }}</span>
									<span class="text-gray-600 text-xs">rating</span>
								</div>

								<div v-if="course.data.enrollment_count" class="flex items-center">
									<Users class="h-3.5 w-3.5 text-gray-500 mr-1" />
									<span class="font-semibold text-gray-900 mr-1 text-xs">{{ course.data.enrollment_count_formatted }}</span>
									<span class="text-gray-600 text-xs">students</span>
								</div>

								<!-- Course Instructors - Ultra Compact -->
								<div class="flex items-center">
									<div class="flex -space-x-1 mr-1.5">
										<UserAvatar
											v-for="instructor in course.data.instructors"
											:key="instructor.name"
											:user="instructor"
											class="w-5 h-5 border-2 border-white"
										/>
									</div>
									<CourseInstructors :instructors="course.data.instructors" class="text-xs" />
								</div>
							</div>

							<!-- Course Tags - Ultra Compact -->
							<div v-if="course.data.tags" class="flex flex-wrap gap-1 mb-3">
								<span
									v-for="tag in course.data.tags.split(', ')"
									:key="tag"
									class="si-badge si-badge-primary text-xs px-2 py-0.5"
								>
									{{ tag }}
								</span>
							</div>
						</div>
					</div>

					<!-- Course Enrollment Card (Desktop Only) -->
					<div class="hidden lg:block lg:col-span-1">
						<div class="si-animate-slideInLeft">
							<CourseCardOverlay :course="course" />
						</div>
					</div>
				</div>

				<!-- Course Enrollment Card (Mobile Only) -->
				<div class="lg:hidden mt-3">
					<CourseCardOverlay :course="course" />
				</div>
			</div>
		</section>
		<!-- Silicon Course Content Section - Compact Layout -->
		<section class="max-w-7xl mx-auto px-6 py-8 lg:py-12">
			<div class="grid lg:grid-cols-3 gap-6 lg:gap-8">
				<!-- Main Content -->
				<div class="lg:col-span-2 space-y-6 lg:space-y-8">
					<!-- Course Description -->
					<div class="si-card">
						<div class="si-card-content">
							<h2 class="text-xl lg:text-2xl font-bold text-gray-900 mb-4 lg:mb-5">{{ __('About This Course') }}</h2>
							<div
								v-html="course.data.description"
								class="prose prose-blue max-w-none prose-headings:text-gray-900 prose-p:text-gray-600 prose-a:text-blue-600 prose-strong:text-gray-900"
							></div>
						</div>
					</div>

					<!-- Course Outline -->
					<div class="si-card">
						<div class="si-card-content">
							<CourseOutline
								:title="__('Course Outline')"
								:courseName="course.data.name"
								:showOutline="true"
							/>
						</div>
					</div>

					<!-- Course Reviews -->
					<CourseReviews
						:courseName="course.data.name"
						:avg_rating="course.data.rating"
						:membership="course.data.membership"
					/>
				</div>

				<!-- Sidebar (Sticky on Desktop) -->
				<div class="lg:col-span-1">
					<div class="sticky top-6 space-y-4 lg:space-y-5">
						<!-- Course Stats Card -->
						<div class="si-card">
							<div class="si-card-content">
								<h3 class="font-semibold text-gray-900 mb-3 lg:mb-4">{{ __('Course Statistics') }}</h3>
								<div class="space-y-2.5">
									<div v-if="course.data.enrollment_count" class="flex justify-between">
										<span class="text-gray-600 text-sm">{{ __('Students Enrolled') }}</span>
										<span class="font-semibold text-sm">{{ course.data.enrollment_count_formatted }}</span>
									</div>
									<div v-if="parseInt(course.data.rating) > 0" class="flex justify-between">
										<span class="text-gray-600 text-sm">{{ __('Average Rating') }}</span>
										<span class="font-semibold text-sm">{{ course.data.rating }} ⭐</span>
									</div>
									<div v-if="course.data.lesson_count" class="flex justify-between">
										<span class="text-gray-600 text-sm">{{ __('Total Lessons') }}</span>
										<span class="font-semibold text-sm">{{ course.data.lesson_count }}</span>
									</div>
									<div v-if="course.data.duration" class="flex justify-between">
										<span class="text-gray-600 text-sm">{{ __('Duration') }}</span>
										<span class="font-semibold text-sm">{{ course.data.duration }}</span>
									</div>
								</div>
							</div>
						</div>

						<!-- Prerequisites Card (if any) -->
						<div v-if="course.data.prerequisites" class="si-card">
							<div class="si-card-content">
								<h3 class="font-semibold text-gray-900 mb-3">{{ __('Prerequisites') }}</h3>
								<div v-html="course.data.prerequisites" class="prose prose-sm text-gray-600"></div>
							</div>
						</div>
					</div>
				</div>
			</div>
		</section>
	</div>
</template>
<script setup>
import {
	createResource,
	Breadcrumbs,
	Badge,
	Tooltip,
	usePageMeta,
} from 'frappe-ui'
import { computed } from 'vue'
import { Users, Star } from 'lucide-vue-next'
import { sessionStore } from '@/stores/session'
import CourseCardOverlay from '@/components/CourseCardOverlay.vue'
import CourseOutline from '@/components/CourseOutline.vue'
import CourseReviews from '@/components/CourseReviews.vue'
import UserAvatar from '@/components/UserAvatar.vue'
import CourseInstructors from '@/components/CourseInstructors.vue'

const { brand } = sessionStore()

const props = defineProps({
	courseName: {
		type: String,
		required: true,
	},
})

const course = createResource({
	url: 'lms.lms.utils.get_course_details',
	cache: ['course', props.courseName],
	params: {
		course: props.courseName,
	},
	auto: true,
})

const breadcrumbs = computed(() => {
	let items = [{ label: 'Courses', route: { name: 'Courses' } }]
	items.push({
		label: course?.data?.title,
		route: { name: 'CourseDetail', params: { courseName: course?.data?.name } },
	})
	return items
})

usePageMeta(() => {
	return {
		title: course?.data?.title,
		icon: brand.favicon,
	}
})
</script>
<style>
.avatar-group {
	display: inline-flex;
	align-items: center;
}

.avatar-group .avatar {
	transition: margin 0.1s ease-in-out;
}
</style>
