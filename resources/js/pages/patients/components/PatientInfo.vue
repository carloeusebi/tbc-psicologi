<script setup lang="ts">
import FormToast from '@/components/FormToast.vue';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import DatePicker from '@/components/ui/date-picker/DatePicker.vue';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Patient, SharedData } from '@/types';
import { router } from '@inertiajs/vue3';
import { getLocalTimeZone, today } from '@internationalized/date';
import { useForm } from 'laravel-precognition-vue-inertia';
import { Contact, Database, Hospital, NotebookTabs } from 'lucide-vue-next';
import { h, onBeforeUnmount, watch } from 'vue';
import { toast } from 'vue-sonner';

const { patient } = defineProps<{
    patient: Patient;
    genders: Array<{ key: string; label: string }>;
}>();

const form = useForm('put', route('patients.update', { patient: patient.id }), { ...patient });

let resolvePromise: (value: string) => void;
let rejectPromise: (reason?: any) => void;

const onFormSubmit = () => {
    if (form.hasErrors) return;

    const promise = new Promise<string>((resolve, reject) => {
        resolvePromise = resolve;
        rejectPromise = reject;
    });

    form.submit({
        preserveScroll: true,
        onStart: () => {
            toast.promise(promise, {
                loading: 'Salvataggio in corso...',
                success: (message: string) => message,
                error: 'Errore durante il salvataggio.',
            });
        },
        onSuccess: (params: Page<SharedData>) => resolvePromise(params.props.flash.success),
        onError: () => rejectPromise(),
    });
};

const dismissFormToast = () => toast.dismiss('form-toast');

watch(
    () => form.isDirty,
    (isDirty) => {
        dismissFormToast();
        if (!isDirty) return;
        toast(h(FormToast, { form, onSubmit: onFormSubmit }), {
            position: 'bottom-center',
            duration: 999999,
            class: '!bg-black !text-white dirty-toast',
            id: 'form-toast',
        });
    },
);

const shakeElement = (el: HTMLElement | null) => {
    if (!el) return;
    el.classList.add('animate-shake');
    el.addEventListener('animationend', () => {
        el.classList.remove('animate-shake');
    });
};

const removeRouterListener = router.on('before', (e) => {
    if (!form.isDirty) return true;
    if (e.detail.visit.method !== 'get') return true;

    if (!e.detail.visit.prefetch) {
        shakeElement(document.querySelector('.dirty-toast'));
    }

    return false;
}) as () => void;

onBeforeUnmount(() => {
    dismissFormToast();
    removeRouterListener();
});

defineExpose({ form });
</script>

<template>
    <form class="grid grid-cols-1 gap-4 md:grid-cols-2" @submit.prevent="onFormSubmit">
        <div>
            <Card>
                <CardHeader>
                    <CardTitle class="flex items-center gap-2">
                        <Contact class="h-5 w-5" />
                        Anagrafica
                    </CardTitle>
                    <CardDescription>Anagrafica del paziente</CardDescription>
                </CardHeader>
                <CardContent class="grid grid-cols-1 gap-2 md:grid-cols-3">
                    <div>
                        <Label for="first_name" :class="{ 'text-red-500': form.invalid('first_name') }">Nome</Label>
                        <Input id="first_name" v-model.ucfirst="form.first_name" @change="form.validate('first_name')" />
                        <div v-if="form.errors.first_name" class="text-sm text-red-500">{{ form.errors.first_name }}</div>
                    </div>
                    <div>
                        <Label for="last_name" :class="{ 'text-red-500': form.invalid('last_name') }">Cognome</Label>
                        <Input id="last_name" v-model.ucfirst="form.last_name" @change="form.validate('last_name')" />
                        <div v-if="form.errors.last_name" class="text-sm text-red-500">{{ form.errors.last_name }}</div>
                    </div>
                    <div>
                        <Label for="birth_date" :class="{ 'text-red-500': form.invalid('last_name') }">Data di Nascita</Label>
                        <DatePicker
                            id="birth_date"
                            v-model="form.birth_date"
                            @update:model-value="form.validate('birth_date')"
                            :max-value="today(getLocalTimeZone())"
                        />
                        <div v-if="form.errors.birth_date" class="text-sm text-red-500">{{ form.errors.birth_date }}</div>
                    </div>
                    <div>
                        <Label for="birth_place" :class="{ 'text-red-500': form.invalid('birth_place') }">Luogo di Nascita</Label>
                        <Input id="birth_place" v-model="form.birth_place" @change="form.validate('birth_place')" />
                        <div v-if="form.errors.birth_place" class="text-sm text-red-500">{{ form.errors.birth_place }}</div>
                    </div>
                    <div>
                        <Label for="gender" :class="{ 'text-red-500': form.invalid('gender') }">Sesso</Label>
                        <Select v-model="form.gender" @update:model-value="form.validate('gender')">
                            <SelectTrigger>
                                <SelectValue placeholder="Seleziona il sesso" />
                            </SelectTrigger>
                            <SelectContent>
                                <SelectItem v-for="gender in genders" :key="gender.key" :value="gender.key">
                                    {{ gender.label }}
                                </SelectItem>
                            </SelectContent>
                        </Select>
                        <div v-if="form.errors.gender" class="text-sm text-red-500">{{ form.errors.gender }}</div>
                    </div>
                    <div>
                        <Label for="codice_fiscale" :class="{ 'text-red-500': form.invalid('codice_fiscale') }">Codice Fiscale</Label>
                        <Input id="codice_fiscale" type="text" v-model.uppercase="form.codice_fiscale" @input="form.validate('codice_fiscale')" />
                        <div v-if="form.errors.codice_fiscale" class="text-sm text-red-500">
                            {{ form.errors.codice_fiscale }}
                        </div>
                    </div>
                </CardContent>
            </Card>
        </div>

        <div>
            <Card>
                <CardHeader>
                    <CardTitle class="flex items-center gap-2">
                        <NotebookTabs class="h-5 w-5" />
                        Contatti
                    </CardTitle>
                    <CardDescription>Contatti e recapiti</CardDescription>
                </CardHeader>
                <CardContent></CardContent>
            </Card>
        </div>

        <div>
            <Card>
                <CardHeader>
                    <CardTitle class="flex items-center gap-2">
                        <Hospital class="h-5 w-5" />
                        Clinica
                    </CardTitle>
                    <CardDescription> Lorem ipsum dolor sit amet consectetur adipisicing elit. Quisquam, quos. </CardDescription>
                </CardHeader>
                <CardContent></CardContent>
            </Card>
        </div>

        <div>
            <Card>
                <CardHeader>
                    <CardTitle class="flex items-center gap-2">
                        <Database class="h-5 w-5" />
                        Record
                    </CardTitle>
                    <CardDescription> Lorem ipsum dolor sit amet consectetur adipisicing elit. Quisquam, quos. </CardDescription>
                </CardHeader>
                <CardContent></CardContent>
            </Card>
        </div>
        <button type="submit" class="hidden" />
    </form>
</template>

<style></style>
