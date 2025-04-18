<script lang="ts">
  import { superForm } from 'sveltekit-superforms';
  import { Field, Control, Label, FieldErrors } from '$lib/components/ui/form';
  import { Input } from '$lib/components/ui/input';
  import * as Card from '$lib/components/ui/card';
  import { z } from 'zod';
  import { zodClient } from 'sveltekit-superforms/adapters';
  import { Button } from '$lib/components/ui/button';
  import { toast } from 'svelte-sonner';
  import { type Infer } from 'sveltekit-superforms/client';
  import { Link } from '$lib/components/ui/link';
  import { login } from '$lib/api';
  import { goto } from '$app/navigation';
  import { base } from '$app/paths';
  import { _ as t } from 'svelte-i18n';

  export const registerSchema = z.object({
    username: z.string().min(2),
    password: z.string().min(6, $t('password-must-be-6-characters-long'))
  });

  export let data: Infer<typeof registerSchema>;

  const form = superForm(data, {
    SPA: true,
    validators: zodClient(registerSchema),
    onUpdated: async ({ form: f }) => {
      if (f.valid) {
        $formData = f.data;
        await login(f.data);
        toast.success($t('login-successful'));
        goto(`${base}/`);
        // toast.success(`You submitted ${JSON.stringify(f.data, null, 2)}`);
      } else {
        toast.error($t('please-fix-the-errors-in-the-form'));
      }
    },
    onSubmit(input) {},

    clearOnSubmit: 'errors',
    multipleSubmits: 'prevent'
  });
  const { form: formData, enhance } = form;
</script>

<!-- <SuperDebug data={$formData} /> -->

<div class="grid h-full w-full place-items-center items-center justify-items-center">
  <form use:enhance on:submit|preventDefault|stopPropagation class="w-full max-w-[500px] space-y-6">
    <Card.Root class="w-full max-w-[500px]">
      <Card.Header>
        <Card.Title>{$t('login')}</Card.Title>
      </Card.Header>
      <Card.Content>
        <Field {form} name="username">
          <Control let:attrs>
            <Label>{$t('username')}</Label>
            <Input {...attrs} bind:value={$formData.username} />
          </Control>
          <FieldErrors />
        </Field>
        <Field {form} name="password">
          <Control let:attrs>
            <Label>{$t('password')}</Label>
            <Input {...attrs} type="password" bind:value={$formData.password} />
          </Control>
          <FieldErrors />
        </Field>
        <Link href="{base}/register">{$t('register-instead')}</Link>
      </Card.Content>
      <Card.Footer>
        <Button type="submit">{$t('login')}</Button>
      </Card.Footer>
    </Card.Root>
  </form>
</div>
