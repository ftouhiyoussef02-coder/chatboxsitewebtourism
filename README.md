import { useState } from "react";
import { MessageCircle, X } from "lucide-react";

export default function ChatButton() {
  const [isOpen, setIsOpen] = useState(false);
  const [messages, setMessages] = useState([
    { from: "bot", text: "Bonjour 👋 Je suis ton assistant IA sécurisé !" },
  ]);
  const [input, setInput] = useState("");

  // Fonction d'envoi du message
  const handleSend = async () => {
    if (!input.trim()) return;

    const newMessages = [...messages, { from: "user", text: input }];
    setMessages(newMessages);
    setInput("");

    try {
      // 🔒 Appel à ton API IA sécurisée sur Hugging Face (remplace "MODEL_NAME" par ton modèle)
      const response = await fetch("https://api-inference.huggingface.co/models/bert-base-uncased", {
        method: "POST",
        headers: {
          "Authorization": "Bearer hf_your_api_key_here", // ⚠️ Mets ici ta clé gratuite Hugging Face
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ inputs: input }),
      });

      const data = await response.json();
      const botReply =
        data?.[0]?.generated_text ||
        "Désolé, je n’ai pas compris. Essaie de reformuler 💬";

      setMessages([...newMessages, { from: "bot", text: botReply }]);
    } catch (error) {
      setMessages([
        ...newMessages,
        { from: "bot", text: "Erreur de connexion à l’IA 😢" },
      ]);
    }
  };

  return (
    <>
      {/* Bouton principal */}
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="fixed bottom-5 right-5 bg-[#002b5b] rounded-full w-[70px] h-[65px] flex items-center justify-center shadow-lg hover:scale-110 transition-transform duration-300 z-50"
        aria-label="Open chat"
      >
        {isOpen ? (
          <X className="w-7 h-7 text-white" />
        ) : (
          <MessageCircle className="w-7 h-7 text-white" />
        )}
      </button>

      {/* Fenêtre de chat */}
      {isOpen && (
        <div className="fixed bottom-24 right-5 w-80 bg-white rounded-2xl shadow-xl border border-gray-200 overflow-hidden z-50">
          <div className="bg-[#002b5b] text-white p-3 font-semibold">
            💬 Chat IA Sécurisé
          </div>

          <div className="h-64 overflow-y-auto p-3 space-y-2">
            {messages.map((msg, index) => (
              <div
                key={index}
                className={`p-2 rounded-lg ${
                  msg.from === "user"
                    ? "bg-blue-100 text-right"
                    : "bg-gray-100 text-left"
                }`}
              >
                {msg.text}
              </div>
            ))}
          </div>

          <div className="flex border-t">
            <input
              type="text"
              value={input}
              onChange={(e) => setInput(e.target.value)}
              className="flex-1 p-2 outline-none"
              placeholder="Écris un message..."
              onKeyDown={(e) => e.key === "Enter" && handleSend()}
            />
            <button
              onClick={handleSend}
              className="bg-[#002b5b] text-white px-4 hover:bg-[#013d7c]"
            >
              ➤
            </button>
          </div>
        </div>
      )}
    </>
  );
}

