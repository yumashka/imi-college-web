<script lang="ts">
	import Main from "@/components/Main.svelte";
	import Button from "@/components/ui/Button.svelte";
	import Input from "@/components/ui/Input.svelte";
	import Label from "@/components/ui/Label.svelte";
	import Select from "@/components/ui/Select.svelte";
	import type { EventHandler } from "svelte/elements";
	import type { PageData } from "./$types";
	import { SetUserAddressSchema } from "@/lib/api-types";
	import { setMyAddress } from "@/lib/api";
	import { goto, invalidateAll } from "$app/navigation";

	export let data: PageData;

	$: dictRegions = data.dictRegions;
	$: dictTownTypes = data.dictTownTypes;

	let regionId: number | undefined = data.address?.regionId;
	let townTypeId: number | undefined = data.address?.townTypeId;
	let address: string | undefined = data.address?.address;
	let town: string | undefined = data.address?.town;
	let postCode: string | undefined = data.address?.postCode;

	let isSubmitting = false;

	const handleSubmit: EventHandler<SubmitEvent, HTMLFormElement> = async () => {
		const body = SetUserAddressSchema.parse({
			regionId,
			townTypeId,
			address,
			town,
			postCode,
		});

		isSubmitting = true;
		try {
			await setMyAddress({ body });
			await invalidateAll();

			goto("/dashboard#address")
		} finally {
			isSubmitting = false;
		}
	};
</script>

<Main class="flex flex-col justify-center">
	<form
		on:submit|preventDefault={handleSubmit}
		class="w-full max-w-screen-md rounded-lg border border-border bg-background p-6"
	>
		<div class="space-y-12">
			<div class="border-b border-border pb-10">
				<h2 class="text-base font-semibold leading-7">Адрес абитуриента</h2>
				<p class="mt-1 text-sm leading-6 text-muted-foreground">Укажите ваше место жительства</p>

				<div class="mt-10 grid grid-cols-1 gap-x-6 gap-y-8 sm:grid-cols-6">
					<div class="sm:col-span-3">
						<Label for="region" required>Регион</Label>
						<div class="mt-2">
							<Select
								bind:value={regionId}
								id="region"
								class="w-full"
								disabled={dictRegions == null}
								required
							>
								<option value={-1} selected disabled>Не выбрано</option>
								{#if dictRegions != null}
									{#each dictRegions as region}
										<option value={region.id}>{region.displayValue || region.value}</option>
									{/each}
								{/if}
							</Select>
						</div>
					</div>

					<div class="sm:col-span-3">
						<Label for="townType" required>Тип населённого пункта</Label>
						<div class="mt-2">
							<Select
								bind:value={townTypeId}
								id="townType"
								class="w-full"
								disabled={dictTownTypes == null}
								required
							>
								<option value={-1} disabled selected>Не выбрано</option>
								{#if dictTownTypes != null}
									{#each dictTownTypes as townType}
										<option value={townType.id}>{townType.displayValue || townType.value}</option>
									{/each}
								{/if}
							</Select>
						</div>
					</div>

					<div class="col-span-full">
						<Label for="street-address" required>Адрес</Label>
						<div class="mt-2">
							<Input
								bind:value={address}
								type="text"
								id="street-address"
								autocomplete="street-address"
								required
							/>
						</div>
					</div>

					<div class="sm:col-span-3 sm:col-start-1">
						<Label for="town" required>Населённый пункт</Label>
						<div class="mt-2">
							<Input
								bind:value={town}
								type="text"
								id="town"
								autocomplete="address-level2"
								required
							/>
						</div>
					</div>
					<div class="sm:col-span-3">
						<Label for="postCode" required>Почтовый индекс</Label>
						<div class="mt-2">
							<Input
								bind:value={postCode}
								type="text"
								id="postCode"
								autocomplete="postCode"
								required
							/>
						</div>
					</div>
				</div>
			</div>
		</div>

		<div class="mt-6 flex items-center justify-end gap-x-4">
			<Button href="/dashboard#address" type="button" variant="ghost" disabled={isSubmitting}>
				Отмена
			</Button>
			<Button type="submit" disabled={isSubmitting}>Сохранить</Button>
		</div>
	</form>
</Main>
