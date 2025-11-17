ll




override suspend fun pullErrorLogs(): Outcome<List<ProductErrorLogEntry>> {
    // If logs aren’t ready, just return empty (or an Error, up to you)
    if (!dynamo.areFaultsAndErrorsReady()) {
        return Outcome.Ok(emptyList())
    }

    val result = dynamo.getErrorLog(ArrayList<String>())

    // 1) Guard against empty list
    if (result.isEmpty()) {
        return Outcome.Ok(emptyList())
    }

    // 2) Handle "No log entries found." message safely
    if (result.size == 1 && result[0].startsWith("No log entries", ignoreCase = true)) {
        return Outcome.Ok(emptyList())
    }

    // 3) Parse each line defensively
    val entries = result.mapNotNull { line ->
        // Expected format: "time - status ..."  (adjust split string if needed)
        val statusSplit = line.split(" - ", limit = 2)
        if (statusSplit.size < 2) return@mapNotNull null

        val time = statusSplit[0]
        val statusPart = statusSplit[1]

        // Example: first token is state code, rest is human-readable status
        val stateSplit = statusPart.split(" ", limit = 2)
        val stateCode = stateSplit[0]
        val status = if (stateSplit.size > 1) stateSplit[1] else ""

        ProductErrorLogEntry(
            code = stateCode,
            status = status,
            state = stateCode,
            timestamp = time
        )
    }.reversed()

    return Outcome.Ok(entries)
}
