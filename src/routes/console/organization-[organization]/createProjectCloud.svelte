<script lang="ts">
    import { Wizard } from '$lib/layout';
    import { sdk } from '$lib/stores/sdk';
    import { onDestroy } from 'svelte';
    import { addNotification } from '$lib/stores/notifications';
    import Step1 from './wizard/step1.svelte';
    import Step2 from './wizard/step2.svelte';
    import type { WizardStepsType } from '$lib/layout/wizard.svelte';
    import { goto, invalidate } from '$app/navigation';
    import { Dependencies } from '$lib/constants';
    import { page } from '$app/stores';
    import { Submit, trackEvent, trackError } from '$lib/actions/analytics';
    import { ID } from '@appwrite.io/console';
    import { createProject } from './wizard/store';
    import { wizard } from '$lib/stores/wizard';

    const teamId = $page.params.organization;

    async function onFinish() {
        await invalidate(Dependencies.FUNCTIONS);
    }

    async function create() {
        try {
            const project = await sdk.forConsole.projects.create(
                $createProject?.id ?? ID.unique(),
                $createProject.name,
                teamId,
                $createProject.region
            );
            trackEvent(Submit.ProjectCreate, {
                customId: !!$createProject?.id,
                teamId,
                region: $createProject.region
            });
            addNotification({
                type: 'success',
                message: `${$createProject.name} has been created`
            });
            await goto(`/console/project-${project.$id}`);
            wizard.hide();
        } catch (e) {
            addNotification({
                type: 'error',
                message: e.mesage
            });
            trackError(e, Submit.ProjectCreate);
        }
    }

    onDestroy(() => {
        $createProject = {
            id: null,
            name: null,
            region: 'eu-de'
        };
    });

    const stepsComponents: WizardStepsType = new Map();
    stepsComponents.set(1, {
        label: 'Project details',
        component: Step1
    });
    stepsComponents.set(2, {
        label: 'Select region',
        component: Step2
    });
</script>

<Wizard title="Create project" steps={stepsComponents} on:finish={create} on:exit={onFinish} />