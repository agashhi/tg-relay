export default {
  async fetch(request, env) {
    if (request.method !== "POST") {
      return new Response("Use POST", { status: 405 });
    }

    const secret = request.headers.get("X-Relay-Secret");
    if (!secret || secret !== env.RELAY_SECRET) {
      return new Response("Unauthorized", { status: 401 });
    }

    const contentType = request.headers.get("Content-Type") || "";
    if (!contentType.toLowerCase().startsWith("multipart/form-data")) {
      return new Response("Expected multipart/form-data", { status: 415 });
    }

    try {
      const incoming = await request.formData();
      const chatId = incoming.get("chat_id");
      const photo = incoming.get("photo");

      if (
        typeof chatId !== "string" ||
        !/^-?\d+$/.test(chatId) ||
        !(photo instanceof File)
      ) {
        return new Response("Missing or invalid chat_id/photo", {
          status: 400
        });
      }

      if (photo.size > 10 * 1024 * 1024) {
        return new Response("Photo exceeds 10 MB", { status: 413 });
      }

      const outgoing = new FormData();
      outgoing.append("chat_id", chatId);
      outgoing.append("photo", photo, photo.name || "screen.png");

      const telegramResponse = await fetch(
        https://api.telegram.org/bot${env.TELEGRAM_BOT_TOKEN}/sendPhoto,
        {
          method: "POST",
          body: outgoing
        }
      );

      const result = await telegramResponse.text();

      return new Response(result, {
        status: telegramResponse.ok ? 200 : 502,
        headers: { "Content-Type": "application/json; charset=utf-8" }
      });
    } catch {
      return new Response("Request processing failed", { status: 400 });
    }
  }
};