<script>
import { Errors } from 'laravel-nova';
import Panel from './Panel.vue';

export default {
    components: {
        Panel
    },
    props: {
        group: String
    },
    computed: {
        title() {
            return this.resources.find(resource => resource.group === this.group)?.title || 'Nova Settings';
        },
        current_resources() {
            return this.resources.filter(resource => resource.group === this.group)
        }
    },
    data() {
        return {
            loading: false,
            resources: [],
            errors: new Errors(),
        };
    },
    async mounted() {
        await this.getFields();
    },
    unmounted() {
        this.resources = [];
        this.activeTab = null;
        this.loading = false;
        this.errors.clear();
    },
    methods: {
        async getFields() {
            try {
                this.loading = true;
                const response = await Nova.request().get(`/nova-vendor/nova-settings/${this.group}`);
                this.resources = response.data.resources;
                this.loading = false;
            } catch (error) {
                this.loading = false;
                Errors.record(error);
            }
        },
        async update() {
            try {
                this.errors.clear();
                const formData = new FormData();
                this.current_resources.forEach(resource => resource.fields.forEach(field => field.fill(formData)));
                formData.append('_method', 'POST');

                await Nova.request().post(`/nova-vendor/nova-settings/${this.group}`, formData);
                this.errors.clear();
                Nova.success('Settings have been saved!');
            } catch (error) {
                if (error?.response?.status === 422 && error?.response?.data?.errors) {
                    this.errors = new Errors(error.response.data.errors || {});
                }

                console.log(error)
            }
        }
    },
}
</script>
<template>
    <div>
        <heading class="mb-6">{{ title }}</heading>

        <card class="nova-settings-form">
            <form @submit.prevent="update">
                <div class="panel-group">
                    <panel
                        v-for="(resource, index) in current_resources"
                        :key="index"
                        :title="resource.title"
                        :description="resource.description"
                        :fields="resource.fields"
                        :errors="errors"
                    />
                </div>

                <div class="flex mt-4 justify-end">
                    <button
                        type="submit"
                        :disabled="loading"
                        class="btn btn-default btn-primary"
                    >
                        Save Settings
                    </button>
                </div>
            </form>
        </card>
    </div>
</template>
