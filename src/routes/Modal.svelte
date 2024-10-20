<script>
    export let show = false;
    export let showInfo = false;
    export let title = '';
    export let onClose = () => {};
</script>
  
  {#if show}
    <div class="modal-overlay" on:click={onClose}>
      <div class="modal" on:click|stopPropagation>
        <h2>{title}</h2>
        <div class="modal-content">
          <slot></slot>
        </div>
      </div>
    </div>
  {/if}

  {#if showInfo}
    <div class="modal-overlay" on:click={onClose}>
      <div class="modal" on:click|stopPropagation>
        <h2>Controls Info</h2>
        <div class="modal-content">
          <slot></slot>
        </div>
      </div>
    </div>
  {/if}
  
  <style>
    .modal-overlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: rgba(0,0,0,0.5);
      display: flex;
      justify-content: center;
      align-items: center;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }
    .modal {
      background-color: white;
      padding: 20px;
      border-radius: 10px;
      width: 300px;
      max-width: 80%;
    }
    .modal-content {
      margin-top: 20px;
    }
  </style>
  