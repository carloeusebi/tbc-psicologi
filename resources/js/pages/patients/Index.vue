<script setup lang="ts">
import { Button } from '@/components/ui/button';
import AppLayout from '@/layouts/AppLayout.vue';
import { BreadcrumbItem, Patient } from '@/types';
import { Head, Link } from '@inertiajs/vue3';
import { columns } from './components/table/columns';
import DataTable from './components/table/DataTable.vue';

defineProps<{
    patients: Array<Patient>;
}>();

const breadcrumbs: BreadcrumbItem[] = [
    {
        title: 'Pazienti',
        href: route('patients.index'),
    },
];
</script>

<template>
    <AppLayout :breadcrumbs>
        <Head title="Pazienti" />
        <div class="h-full max-w-[100vw] flex-1 flex-col space-y-8 p-2 md:flex md:p-4 lg:p-8">
            <div class="items-center justify-between space-y-2 md:flex">
                <div>
                    <h2 class="text-2xl font-bold tracking-tight">Pazienti</h2>
                    <p class="text-muted-foreground">Ecco una lista dei pazienti attivi</p>
                </div>
                <div class="flex items-center space-x-2">
                    <Button :as="Link" :href="route('patients.create')"> Aggiungi Paziente</Button>
                </div>
            </div>
            <DataTable :data="patients" :columns="columns" />
        </div>
    </AppLayout>
</template>
