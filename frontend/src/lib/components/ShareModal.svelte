<script lang="ts">
	import type { Collection, User } from '$lib/types';
	import { createEventDispatcher } from 'svelte';
	const dispatch = createEventDispatcher();
	import { onMount } from 'svelte';
	import UserCard from './UserCard.svelte';
	import { addToast } from '$lib/toasts';
	let modal: HTMLDialogElement;
	import { t } from 'svelte-i18n';

	export let collection: Collection;

	let allUsers: User[] = [];

	let sharedWithUsers: User[] = [];
	let notSharedWithUsers: User[] = [];

	async function share(user: User) {
		let res = await fetch(`/api/collections/${collection.id}/share/${user.uuid}/`, {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			}
		});
		if (res.ok) {
			sharedWithUsers = sharedWithUsers.concat(user);
			if (collection.shared_with) {
				collection.shared_with.push(user.uuid);
			} else {
				collection.shared_with = [user.uuid];
			}
			notSharedWithUsers = notSharedWithUsers.filter((u) => u.uuid !== user.uuid);
			addToast(
				'success',
				`${$t('share.shared')} ${collection.name} ${$t('share.with')} ${user.first_name} ${user.last_name}`
			);
		}
	}

	async function unshare(user: User) {
		let res = await fetch(`/api/collections/${collection.id}/unshare/${user.uuid}/`, {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			}
		});
		if (res.ok) {
			notSharedWithUsers = notSharedWithUsers.concat(user);
			if (collection.shared_with) {
				collection.shared_with = collection.shared_with.filter((u) => u !== user.uuid);
			}
			sharedWithUsers = sharedWithUsers.filter((u) => u.uuid !== user.uuid);
			addToast(
				'success',
				`${$t('share.unshared')} ${collection.name} ${$t('share.with')} ${user.first_name} ${user.last_name}`
			);
		}
	}

	onMount(async () => {
		modal = document.getElementById('my_modal_1') as HTMLDialogElement;
		if (modal) {
			modal.showModal();
		}
		let res = await fetch(`/auth/users`);
		if (res.ok) {
			let data = await res.json();
			allUsers = data;
			sharedWithUsers = allUsers.filter((user) =>
				(collection.shared_with ?? []).includes(user.uuid)
			);
			notSharedWithUsers = allUsers.filter(
				(user) => !(collection.shared_with ?? []).includes(user.uuid)
			);
			console.log(sharedWithUsers);
			console.log(notSharedWithUsers);
		}
	});

	function close() {
		dispatch('close');
	}

	function handleKeydown(event: KeyboardEvent) {
		if (event.key === 'Escape') {
			dispatch('close');
		}
	}
</script>

<dialog id="my_modal_1" class="modal">
	<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
	<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
	<div class="modal-box w-11/12 max-w-5xl" role="dialog" on:keydown={handleKeydown} tabindex="0">
		<h3 class="font-bold text-lg">{$t('adventures.share')} {collection.name}</h3>
		<p class="py-1">{$t('share.share_desc')}</p>
		<div class="divider"></div>
		<h3 class="font-bold text-md">{$t('share.shared_with')}</h3>
		<ul>
			{#each sharedWithUsers as user}
				<div class="flex flex-wrap gap-4 mr-4 justify-center content-center">
					<UserCard
						{user}
						shared_with={collection.shared_with}
						sharing={true}
						on:share={(event) => share(event.detail)}
						on:unshare={(event) => unshare(event.detail)}
					/>
				</div>
			{/each}
			{#if sharedWithUsers.length === 0}
				<p class="text-neutral-content">{$t('share.no_users_shared')}</p>
			{/if}
		</ul>
		<div class="divider"></div>
		<h3 class="font-bold text-md">{$t('share.not_shared_with')}</h3>
		<ul>
			{#each notSharedWithUsers as user}
				<div class="flex flex-wrap gap-4 mr-4 justify-center content-center">
					<UserCard
						{user}
						shared_with={collection.shared_with}
						sharing={true}
						on:share={(event) => share(event.detail)}
						on:unshare={(event) => unshare(event.detail)}
					/>
				</div>
			{/each}
			{#if notSharedWithUsers.length === 0}
				<p class="text-neutral-content">{$t('share.no_users_shared')}</p>
			{/if}
		</ul>
		<button class="btn btn-primary mt-4" on:click={close}>{$t('about.close')}</button>
	</div>
</dialog>
