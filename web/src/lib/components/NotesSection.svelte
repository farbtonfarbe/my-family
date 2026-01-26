<script lang="ts">
	import { api, type Note } from '$lib/api/client';

	interface Props {
		notes?: string;
		noteIds?: string[];
	}

	let { notes, noteIds = [] }: Props = $props();

	let linkedNotes: Note[] = $state([]);
	let loading = $state(false);
	let error: string | null = $state(null);

	async function loadLinkedNotes() {
		if (!noteIds || noteIds.length === 0) {
			linkedNotes = [];
			return;
		}

		loading = true;
		error = null;
		try {
			linkedNotes = await api.getNotesByIds(noteIds);
		} catch (e) {
			error = (e as { message?: string }).message || 'Failed to load linked notes';
		} finally {
			loading = false;
		}
	}

	$effect(() => {
		if (noteIds && noteIds.length > 0) {
			loadLinkedNotes();
		} else {
			linkedNotes = [];
		}
	});

	const hasContent = $derived(!!notes || noteIds.length > 0);
</script>

{#if hasContent}
	<div class="notes-section">
		<h2>Notes</h2>

		{#if error}
			<div class="section-error" role="alert">{error}</div>
		{/if}

		{#if notes}
			<div class="inline-notes">
				<p class="note-text">{notes}</p>
			</div>
		{/if}

		{#if loading}
			<div class="loading-state" role="status" aria-live="polite">Loading linked notes...</div>
		{:else if linkedNotes.length > 0}
			<div class="linked-notes">
				{#each linkedNotes as note}
					<div class="note-card">
						<p class="note-text">{note.text}</p>
						{#if note.gedcom_xref}
							<span class="note-ref">{note.gedcom_xref}</span>
						{/if}
					</div>
				{/each}
			</div>
		{/if}
	</div>
{/if}

<style>
	.notes-section {
		margin-top: 1.5rem;
	}

	.notes-section h2 {
		margin: 0 0 1rem;
		font-size: 0.875rem;
		font-weight: 600;
		color: #64748b;
		text-transform: uppercase;
		letter-spacing: 0.05em;
	}

	.section-error {
		padding: 0.75rem;
		background: #fef2f2;
		border: 1px solid #fecaca;
		border-radius: 6px;
		color: #dc2626;
		font-size: 0.875rem;
		margin-bottom: 1rem;
	}

	.loading-state {
		text-align: center;
		padding: 1rem;
		color: #64748b;
		font-size: 0.875rem;
	}

	.inline-notes {
		margin-bottom: 1rem;
	}

	.note-text {
		margin: 0;
		font-size: 0.9375rem;
		color: #334155;
		line-height: 1.6;
		white-space: pre-wrap;
	}

	.linked-notes {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
	}

	.note-card {
		padding: 1rem;
		background: #f8fafc;
		border: 1px solid #e2e8f0;
		border-radius: 8px;
	}

	.note-card .note-text {
		margin-bottom: 0.5rem;
	}

	.note-ref {
		display: inline-block;
		padding: 0.125rem 0.5rem;
		background: #e2e8f0;
		border-radius: 4px;
		font-size: 0.6875rem;
		color: #64748b;
		font-family: monospace;
	}
</style>
