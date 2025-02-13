const API_TOKEN = "your_cloudflare_api_token"; // Replace with your Cloudflare API token
const ZONE_ID = "your_zone_id"; // Replace with your Zone ID

// Map subdomains to their DNS Record IDs
const DNS_RECORDS = {
    "wan1.yourdomain.com": "your_dns_record_id_for_wan1",
    "wan2.yourdomain.com": "your_dns_record_id_for_wan2",
};

const USERNAME = "your_username"; // Your DDNS username
const PASSWORD = "your_password"; // Your DDNS password

async function handleRequest(request) {
    const authHeader = request.headers.get("Authorization");
    if (!authHeader || !authHeader.startsWith("Basic ")) {
        return new Response("Unauthorized", { status: 401 });
    }

    const encodedCredentials = authHeader.replace("Basic ", "");
    const decodedCredentials = atob(encodedCredentials);
    const [username, password] = decodedCredentials.split(":");

    if (username !== USERNAME || password !== PASSWORD) {
        return new Response("Unauthorized", { status: 401 });
    }

    const url = new URL(request.url);
    const hostname = url.searchParams.get("hostname");
    if (!hostname) {
        return new Response(JSON.stringify({ success: false, error: "Hostname missing" }), {
            status: 400,
            headers: { "Content-Type": "application/json" },
        });
    }

    const dnsRecordId = DNS_RECORDS[hostname];
    if (!dnsRecordId) {
        return new Response(JSON.stringify({ success: false, error: "Hostname not found" }), {
            status: 404,
            headers: { "Content-Type": "application/json" },
        });
    }

    const clientIP = request.headers.get("CF-Connecting-IP");
    if (!clientIP) {
        return new Response(JSON.stringify({ success: false, error: "Could not determine client IP" }), {
            status: 500,
            headers: { "Content-Type": "application/json" },
        });
    }

    const updateURL = `https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${dnsRecordId}`;
    const updateBody = JSON.stringify({
        type: "A",
        name: hostname,
        content: clientIP,
        ttl: 3600,
        proxied: false,
    });

    const updateResponse = await fetch(updateURL, {
        method: "PUT",
        headers: {
            "Authorization": `Bearer ${API_TOKEN}`,
            "Content-Type": "application/json",
        },
        body: updateBody,
    });

    const updateResult = await updateResponse.json();

    if (updateResult.success) {
        return new Response(JSON.stringify({ success: true, message: `DNS record for ${hostname} updated`, clientIP }), {
            status: 200,
            headers: { "Content-Type": "application/json" },
    });
    } else {
        return new Response(JSON.stringify({ success: false, error: updateResult.errors }), {
            status: 500,
        headers: { "Content-Type": "application/json" },
    });
    }
}

addEventListener("fetch", event => {
    event.respondWith(handleRequest(event.request));
});
