<script>
    let currentTime = new Date();

    // Update time every second
    setInterval(() => {
        currentTime = new Date();
    }, 1000);
</script>

<div>
    <p>Current Time: {currentTime.toLocaleString()}</p>
</div>