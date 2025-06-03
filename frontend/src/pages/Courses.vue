<template>
	<!-- Silicon Navigation Header -->
	<header class="si-nav">
		<div class="si-nav-content">
			<Breadcrumbs :items="breadcrumbs" />
			<router-link
				v-if="canCreateCourse()"
				:to="{
					name: 'CourseForm',
					params: { courseName: 'new' },
				}"
			>
				<Button variant="solid" class="si-btn si-btn-primary">
					<template #prefix>
						<Plus class="h-4 w-4 stroke-1.5" />
					</template>
					{{ __('New Course') }}
				</Button>
			</router-link>
		</div>
	</header>

	<!-- Silicon Hero Section -->
	<section class="si-hero">
		<div class="si-hero-content">
			<h1 class="si-hero-title si-animate-fadeInUp">
				{{ __('Discover Amazing Courses') }}
			</h1>
			<p class="si-hero-subtitle si-animate-fadeInUp">
				{{ __('Explore our comprehensive collection of courses designed to help you learn new skills and advance your career.') }}
			</p>
		</div>
	</section>

	<!-- Silicon Search Container -->
	<div class="max-w-7xl mx-auto px-6 -mt-16 relative z-10">
		<div class="si-search-container si-animate-slideInUp">
			<!-- Filter Tabs -->
			<div class="si-filter-tabs">
				<TabButtons :buttons="courseTabs" v-model="currentTab" />
			</div>
			
			<!-- Search and Filters Row -->
			<div class="flex flex-col lg:flex-row gap-4 items-start lg:items-center">
				<!-- Search Input -->
				<div class="flex-1 w-full">
					<FormControl
						v-model="title"
						:placeholder="__('Search courses...')"
						type="text"
						class="si-search-input"
						@input="updateCourses()"
					/>
				</div>
				
				<!-- Filter Controls -->
				<div class="flex flex-col sm:flex-row gap-3 w-full lg:w-auto">
					<Select
						v-if="categories.length"
						v-model="currentCategory"
						:options="categories"
						:placeholder="__('All Categories')"
						@change="updateCourses()"
						class="min-w-40"
					/>
					
					<div class="flex items-center space-x-2">
						<FormControl
							v-model="certification"
							:label="__('Certified Only')"
							type="checkbox"
							@change="updateCourses()"
							class="flex items-center"
						/>
					</div>
				</div>
			</div>
		</div>
	</div>

	<!-- Silicon Course Grid Section -->
	<div class="max-w-7xl mx-auto px-6 pb-20">
		<!-- Courses Grid -->
		<div
			v-if="courses.data?.length"
			class="si-course-grid si-animate-fadeInUp"
		>
			<router-link
				v-for="course in courses.data"
				:key="course.name"
				:to="{ name: 'CourseDetail', params: { courseName: course.name } }"
				class="block transform transition-all duration-300 hover:scale-105"
			>
				<CourseCard :course="course" />
			</router-link>
		</div>

		<!-- Silicon Empty State -->
		<div
			v-else-if="!courses.list.loading"
			class="text-center py-20 si-animate-fadeInUp"
		>
			<div class="si-card max-w-lg mx-auto">
				<div class="si-card-content text-center py-16">
					<div class="w-20 h-20 mx-auto mb-6 bg-gradient-to-br from-blue-50 to-indigo-100 rounded-full flex items-center justify-center">
						<BookOpen class="w-10 h-10 text-blue-500 stroke-1" />
					</div>
					<h3 class="text-2xl font-bold text-gray-900 mb-4">
						{{ __('No courses found') }}
					</h3>
					<p class="text-gray-600 leading-relaxed mb-6">
						{{
							__(
								'There are no courses matching your criteria. Keep an eye out, fresh learning experiences are on the way soon!'
							)
						}}
					</p>
					<Button @click="() => { title = ''; currentCategory = null; certification = false; updateCourses(); }" 
						class="si-btn si-btn-outline">
						{{ __('Clear Filters') }}
					</Button>
				</div>
			</div>
		</div>

		<!-- Loading State -->
		<div
			v-if="courses.list.loading && !courses.data?.length"
			class="si-course-grid"
		>
			<div v-for="i in 6" :key="i" class="si-card animate-pulse">
				<div class="h-48 bg-gray-200 rounded-t-xl"></div>
				<div class="si-card-content">
					<div class="h-4 bg-gray-200 rounded mb-3"></div>
					<div class="h-3 bg-gray-200 rounded mb-2 w-3/4"></div>
					<div class="h-3 bg-gray-200 rounded w-1/2"></div>
				</div>
			</div>
		</div>

		<!-- Silicon Load More Button -->
		<div
			v-if="!courses.list.loading && courses.hasNextPage"
			class="flex justify-center mt-12 si-animate-fadeInUp"
		>
			<Button @click="courses.next()" class="si-btn si-btn-outline hover:si-btn-primary">
				{{ __('Load More Courses') }}
			</Button>
		</div>
	</div>
</template>
<script setup>
import {
	Breadcrumbs,
	Button,
	call,
	createListResource,
	FormControl,
	Select,
	TabButtons,
	usePageMeta,
} from 'frappe-ui'
import { computed, inject, onMounted, ref, watch } from 'vue'
import { BookOpen, Plus } from 'lucide-vue-next'
import { sessionStore } from '@/stores/session'
import { canCreateCourse } from '@/utils'
import CourseCard from '@/components/CourseCard.vue'
import router from '../router'

const user = inject('$user')
const dayjs = inject('$dayjs')
const start = ref(0)
const pageLength = ref(30)
const categories = ref([])
const currentCategory = ref(null)
const title = ref('')
const certification = ref(false)
const filters = ref({})
const currentTab = ref('Live')
const { brand } = sessionStore()

onMounted(() => {
	identifyUserPersona()
	setFiltersFromQuery()
	updateCourses()
	categories.value = [
		{
			label: '',
			value: null,
		},
	]
})

const setFiltersFromQuery = () => {
	let queries = new URLSearchParams(location.search)
	title.value = queries.get('title') || ''
	currentCategory.value = queries.get('category') || null
	certification.value = queries.get('certification') || false
}

const courses = createListResource({
	doctype: 'LMS Course',
	url: 'lms.lms.utils.get_courses',
	cache: ['courses', user.data?.name],
	pageLength: pageLength.value,
	start: start.value,
	onSuccess(data) {
		setCategories(data)
	},
})

const setCategories = (data) => {
	let allCategories = data.map((course) => course.category)
	allCategories = allCategories.filter(
		(category, index) => allCategories.indexOf(category) === index && category
	)
	if (categories.value.length <= allCategories.length) {
		updateCategories(data)
	}
}

const isPersonaCaptured = async () => {
	let persona = await call('frappe.client.get_single_value', {
		doctype: 'LMS Settings',
		field: 'persona_captured',
	})
	return persona
}

const identifyUserPersona = async () => {
	if (user.data?.is_system_manager && !user.data?.developer_mode) {
		let personaCaptured = await isPersonaCaptured()
		if (personaCaptured) return

		call('frappe.client.get_count', {
			doctype: 'LMS Course',
		}).then((data) => {
			if (!data) {
				router.push({
					name: 'PersonaForm',
				})
			}
		})
	}
}

const updateCourses = () => {
	updateFilters()
	courses.update({
		filters: filters.value,
	})
	courses.reload()
}

const updateFilters = () => {
	updateCategoryFilter()
	updateTitleFilter()
	updateCertificationFilter()
	updateTabFilter()
	updateStudentFilter()
	setQueryParams()
}

const updateCategoryFilter = () => {
	if (currentCategory.value) {
		filters.value['category'] = currentCategory.value
	} else {
		delete filters.value['category']
	}
}

const updateTitleFilter = () => {
	if (title.value) {
		filters.value['title'] = ['like', `%${title.value}%`]
	} else {
		delete filters.value['title']
	}
}

const updateCertificationFilter = () => {
	if (certification.value) {
		filters.value['certification'] = 1
	} else {
		delete filters.value['certification']
	}
}

const updateTabFilter = () => {
	delete filters.value['live']
	delete filters.value['created']
	delete filters.value['published_on']
	delete filters.value['upcoming']

	if (currentTab.value == 'Enrolled' && user.data?.is_student) {
		filters.value['enrolled'] = 1
		delete filters.value['published']
	} else {
		delete filters.value['published']
		delete filters.value['enrolled']

		if (currentTab.value == 'Live') {
			filters.value['published'] = 1
			filters.value['upcoming'] = 0
			filters.value['live'] = 1
		} else if (currentTab.value == 'Upcoming') {
			filters.value['upcoming'] = 1
			filters.value['published'] = 1
		} else if (currentTab.value == 'New') {
			filters.value['published'] = 1
			filters.value['published_on'] = [
				'>=',
				dayjs().add(-3, 'month').format('YYYY-MM-DD'),
			]
		} else if (currentTab.value == 'Created') {
			filters.value['created'] = 1
		}
	}
}

const updateStudentFilter = () => {
	if (!user.data || (user.data?.is_student && currentTab.value != 'Enrolled')) {
		filters.value['published'] = 1
	}
}

const setQueryParams = () => {
	let queries = new URLSearchParams(location.search)
	let filterKeys = {
		title: title.value,
		category: currentCategory.value,
		certification: certification.value,
	}

	Object.keys(filterKeys).forEach((key) => {
		if (filterKeys[key]) {
			queries.set(key, filterKeys[key])
		} else {
			queries.delete(key)
		}
	})

	let queryString = ''
	if (queries.toString()) {
		queryString = `?${queries.toString()}`
	}

	history.replaceState({}, '', `${location.pathname}${queryString}`)
}

const updateCategories = (data) => {
	data.forEach((course) => {
		if (
			course.category &&
			!categories.value.find((category) => category.value === course.category)
		)
			categories.value.push({
				label: course.category,
				value: course.category,
			})
	})
}

watch(currentTab, () => {
	updateCourses()
})

const courseTabs = computed(() => {
	let tabs = [
		{
			label: __('Live'),
		},
		{
			label: __('New'),
		},
		{
			label: __('Upcoming'),
		},
	]
	if (
		user.data?.is_moderator ||
		user.data?.is_instructor ||
		user.data?.is_evaluator
	) {
		tabs.push({ label: __('Created') })
	} else if (user.data) {
		tabs.push({ label: __('Enrolled') })
	}
	return tabs
})

const breadcrumbs = computed(() => [
	{
		label: __('Courses'),
		route: { name: 'Courses' },
	},
])

usePageMeta(() => {
	return {
		title: __('Courses'),
		icon: brand.favicon,
	}
})
</script>
