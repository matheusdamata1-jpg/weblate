<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>DAMTECH Engenharia - Sistema Operacional</title>
  
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- React & ReactDOM CDN -->
  <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
  
  <!-- Babel CDN para compilar JSX no navegador -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  
  <!-- Supabase JS Client CDN -->
  <script src="https://unpkg.com/@supabase/supabase-js@2"></script>
</head>
<body class="bg-slate-100 font-sans text-slate-800 antialiased">

  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect } = React;

    // --- CONFIGURAÇÃO SUPABASE (Substitua se desejar conectar ao seu banco de dados) ---
    const SUPABASE_URL = "https://sua-url-supabase.supabase.co";
    const SUPABASE_KEY = "sua-chave-anonima-supabase";
    const supabase = window.supabase ? window.supabase.createClient(SUPABASE_URL, SUPABASE_KEY) : null;

    // --- DADOS INICIAIS DE DEMONSTRAÇÃO ---
    const OS_INICIAIS = [
      { id: "1", codigo_os: 1001, cliente: "Hospital Central", unidade: "Bloco Cirúrgico", tipo: "Preventiva", status: "Aberta", equipamento: "Chiller Carrier 50TR", data: "2026-10-05" },
      { id: "2", codigo_os: 1002, cliente: "Shopping Plaza", unidade: "Praça de Alimentação", tipo: "Corretiva", status: "Em Atendimento", equipamento: "Self Contained 15TR", data: "2026-10-05" },
      { id: "3", codigo_os: 1003, cliente: "Edifício Infinity", unidade: "Andar 12", tipo: "PMOC", status: "Concluída", equipamento: "Split Hi-Wall 18000 BTU", data: "2026-10-04" }
    ];

    const ESTOQUE_INICIAL = [
      { id: "1", codigo: "GAS-R410A", nome: "Gás Refrigerante R410A (Botijão 11.3kg)", qtd: 8, unidade: "kg" },
      { id: "2", codigo: "FIL-G4-20", nome: "Filtro de Ar Moldura G4 500x500mm", qtd: 2, unidade: "un" },
      { id: "3", codigo: "DISJ-3P-32A", nome: "Disjuntor Tripolar DIN 32A", qtd: 15, unidade: "un" }
    ];

    function App() {
      const [aba, setAba] = useState('dashboard');
      const [osLista, setOsLista] = useState(OS_INICIAIS);
      const [estoque, setEstoque] = useState(ESTOQUE_INICIAL);
      
      // Form Nova OS
      const [novaOS, setNovaOS] = useState({ cliente: '', unidade: '', equipamento: '', tipo: 'Preventiva', descricao: '' });
      
      // Ponto Eletrônico
      const [gps, setGps] = useState(null);
      const [pontosHoje, setPontosHoje] = useState([]);

      // Execução de OS
      const [osAtiva, setOsAtiva] = useState(null);
      const [diagnostico, setDiagnostico] = useState('');
      const [solucao, setSolucao] = useState('');
      const [recebedor, setRecebedor] = useState('');

      useEffect(() => {
        if ('geolocation' in navigator) {
          navigator.geolocation.getCurrentPosition(
            (pos) => setGps({ lat: pos.coords.latitude, lng: pos.coords.longitude }),
            (err) => console.warn("GPS Indisponível:", err.message)
          );
        }
      }, []);

      // Ações
      const handleCriarOS = (e) => {
        e.preventDefault();
        const nova = {
          id: Date.now().toString(),
          codigo_os: 1000 + osLista.length + 1,
          cliente: novaOS.cliente || "Cliente Exemplo",
          unidade: novaOS.unidade || "Sede Principal",
          tipo: novaOS.tipo,
          status: "Aberta",
          equipamento: novaOS.equipamento || "Sistema de Climatização",
          data: new Date().toISOString().split('T')[0]
        };
        setOsLista([nova, ...osLista]);
        setNovaOS({ cliente: '', unidade: '', equipamento: '', tipo: 'Preventiva', descricao: '' });
        setAba('os');
      };

      const handleRegistrarPonto = (tipo) => {
        const hora = new Date().toLocaleTimeString('pt-BR');
        setPontosHoje([...pontosHoje, { tipo, hora, lat: gps?.lat, lng: gps?.lng }]);
      };

      const handleFinalizarOS = () => {
        if (!osAtiva) return;
        setOsLista(osLista.map(o => o.id === osAtiva.id ? { ...o, status: 'Concluída' } : o));
        setOsAtiva(null);
        setDiagnostico('');
        setSolucao('');
        setRecebedor('');
        alert('Ordem de Serviço encerrada com sucesso!');
      };

      return (
        <div className="min-h-screen flex flex-col">
          
          {/* TOPO DA APLICAÇÃO */}
          <header className="bg-slate-900 text-white sticky top-0 z-50 border-b border-slate-800 shadow-lg">
            <div className="max-w-7xl mx-auto px-4 py-3 flex justify-between items-center">
              <div className="flex items-center space-x-3">
                <div className="bg-amber-500 text-slate-950 font-black px-3 py-1.5 rounded-lg text-lg tracking-wider">
                  DAMTECH
                </div>
                <div>
                  <h1 className="font-bold text-sm tracking-wide">DAMTECH ENGENHARIA</h1>
                  <p className="text-[10px] text-slate-400">Sistema Operacional de Manutenção</p>
                </div>
              </div>
              <span className="text-xs bg-slate-800 text-amber-400 px-3 py-1 rounded-full font-mono border border-slate-700">
                🟢 Sistema Operacional
              </span>
            </div>

            {/* NAVEGAÇÃO */}
            <nav className="bg-slate-950 px-4 flex space-x-1 overflow-x-auto text-xs border-t border-slate-800">
              {[
                { id: 'dashboard', label: '📊 Dashboard' },
                { id: 'os', label: '🔧 Ordens de Serviço' },
                { id: 'ponto', label: '⏱️ Ponto GPS' },
                { id: 'estoque', label: '📦 Estoque' },
                { id: 'pmoc', label: '🛡️ PMOC' },
              ].map((item) => (
                <button
                  key={item.id}
                  onClick={() => setAba(item.id)}
                  className={`px-4 py-3 font-semibold whitespace-nowrap border-b-2 transition ${
                    aba === item.id 
                      ? 'border-amber-500 text-amber-500 bg-slate-900' 
                      : 'border-transparent text-slate-400 hover:text-slate-200'
                  }`}
                >
                  {item.label}
                </button>
              ))}
            </nav>
          </header>

          {/* CONTEÚDO PRINCIPAL */}
          <main className="flex-1 max-w-7xl w-full mx-auto p-4 md:p-6">

            {/* DASHBOARD */}
            {aba === 'dashboard' && (
              <div className="space-y-6">
                <div className="grid grid-cols-2 lg:grid-cols-4 gap-4">
                  <div className="bg-white p-4 rounded-xl border border-slate-200 shadow-sm border-l-4 border-l-amber-500">
                    <p className="text-xs font-bold text-slate-400 uppercase">OS Em Aberto</p>
                    <p className="text-3xl font-black text-slate-800 mt-1">{osLista.filter(o => o.status !== 'Concluída').length}</p>
                  </div>
                  <div className="bg-white p-4 rounded-xl border border-slate-200 shadow-sm border-l-4 border-l-emerald-500">
                    <p className="text-xs font-bold text-slate-400 uppercase">OS Concluídas</p>
                    <p className="text-3xl font-black text-slate-800 mt-1">{osLista.filter(o => o.status === 'Concluída').length}</p>
                  </div>
                  <div className="bg-white p-4 rounded-xl border border-slate-200 shadow-sm border-l-4 border-l-rose-500">
                    <p className="text-xs font-bold text-slate-400 uppercase">Estoque Crítico</p>
                    <p className="text-3xl font-black text-slate-800 mt-1">{estoque.filter(e => e.qtd < 5).length}</p>
                  </div>
                  <div className="bg-white p-4 rounded-xl border border-slate-200 shadow-sm border-l-4 border-l-blue-500">
                    <p className="text-xs font-bold text-slate-400 uppercase">Equipamentos PMOC</p>
                    <p className="text-3xl font-black text-slate-800 mt-1">24</p>
                  </div>
                </div>

                <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                  {/* Formulário de Abertura de OS */}
                  <div className="bg-white p-5 rounded-xl border border-slate-200 shadow-sm">
                    <h3 className="font-bold text-slate-800 mb-3 flex items-center gap-2">➕ Abrir Nova Ordem de Serviço</h3>
                    <form onSubmit={handleCriarOS} className="space-y-3">
                      <div>
                        <label className="text-xs font-semibold text-slate-600 block">Cliente / Razão Social</label>
                        <input type="text" required placeholder="Ex: Hospital São Lucas" className="w-full mt-1 p-2 border rounded-lg text-sm bg-slate-50" value={novaOS.cliente} onChange={e => setNovaOS({...novaOS, cliente: e.target.value})} />
                      </div>
                      <div className="grid grid-cols-2 gap-2">
                        <div>
                          <label className="text-xs font-semibold text-slate-600 block">Unidade / Setor</label>
                          <input type="text" placeholder="Ex: Bloco A" className="w-full mt-1 p-2 border rounded-lg text-sm bg-slate-50" value={novaOS.unidade} onChange={e => setNovaOS({...novaOS, unidade: e.target.value})} />
                        </div>
                        <div>
                          <label className="text-xs font-semibold text-slate-600 block">Tipo</label>
                          <select className="w-full mt-1 p-2 border rounded-lg text-sm bg-slate-50" value={novaOS.tipo} onChange={e => setNovaOS({...novaOS, tipo: e.target.value})}>
                            <option value="Preventiva">Preventiva</option>
                            <option value="Corretiva">Corretiva</option>
                            <option value="PMOC">PMOC</option>
                            <option value="Instalação">Instalação</option>
                          </select>
                        </div>
                      </div>
                      <div>
                        <label className="text-xs font-semibold text-slate-600 block">Equipamento / Ativo</label>
                        <input type="text" placeholder="Ex: Chiller 01 / FanCoil" className="w-full mt-1 p-2 border rounded-lg text-sm bg-slate-50" value={novaOS.equipamento} onChange={e => setNovaOS({...novaOS, equipamento: e.target.value})} />
                      </div>
                      <button type="submit" className="w-full py-2.5 bg-slate-900 hover:bg-slate-800 text-white font-bold text-sm rounded-lg transition shadow">
                        Cadastrar Ordem de Serviço
                      </button>
                    </form>
                  </div>

                  {/* Lista de OS Recentes */}
                  <div className="bg-white p-5 rounded-xl border border-slate-200 shadow-sm flex flex-col justify-between">
                    <div>
                      <h3 className="font-bold text-slate-800 mb-3">📋 Últimas OS Cadastradas</h3>
                      <div className="space-y-2">
                        {osLista.slice(0, 4).map((os) => (
                          <div key={os.id} className="p-3 bg-slate-50 rounded-lg border border-slate-200 flex justify-between items-center text-xs">
                            <div>
                              <p className="font-bold text-slate-800">#{os.codigo_os} - {os.cliente}</p>
                              <p className="text-slate-500">{os.equipamento} ({os.tipo})</p>
                            </div>
                            <span className={`px-2.5 py-1 rounded-full font-bold uppercase text-[10px] ${os.status === 'Concluída' ? 'bg-emerald-100 text-emerald-800' : 'bg-amber-100 text-amber-800'}`}>
                              {os.status}
                            </span>
                          </div>
                        ))}
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            )}

            {/* ORDENS DE SERVIÇO */}
            {aba === 'os' && (
              <div>
                {osAtiva ? (
                  /* ATENDIMENTO DA OS */
                  <div className="bg-white p-6 rounded-xl border border-slate-200 shadow-sm max-w-2xl mx-auto space-y-4">
                    <div className="flex justify-between items-center border-b pb-3">
                      <div>
                        <h2 className="text-lg font-bold text-slate-800">Executando OS #{osAtiva.codigo_os}</h2>
                        <p className="text-xs text-slate-500">{osAtiva.cliente} — {osAtiva.equipamento}</p>
                      </div>
                      <button onClick={() => setOsAtiva(null)} className="text-xs text-slate-500 hover:underline">← Cancelar</button>
                    </div>

                    <div>
                      <label className="text-xs font-semibold block text-slate-700">Diagnóstico Técnico</label>
                      <textarea rows="3" className="w-full mt-1 p-2 border rounded-lg text-sm" placeholder="Relate o diagnóstico e os testes efetuados..." value={diagnostico} onChange={e => setDiagnostico(e.target.value)}></textarea>
                    </div>

                    <div>
                      <label className="text-xs font-semibold block text-slate-700">Solução Aplicada / Peças Utilizadas</label>
                      <textarea rows="3" className="w-full mt-1 p-2 border rounded-lg text-sm" placeholder="Descreva os procedimentos corretivos realizados..." value={solucao} onChange={e => setSolucao(e.target.value)}></textarea>
                    </div>

                    <div className="grid grid-cols-2 gap-3 bg-slate-50 p-3 rounded-lg border border-dashed border-slate-300 text-center">
                      <div>
                        <p className="text-xs font-bold text-slate-600 mb-1">📸 Foto Antes</p>
                        <input type="file" capture="environment" accept="image/*" className="text-xs w-full" />
                      </div>
                      <div>
                        <p className="text-xs font-bold text-slate-600 mb-1">📸 Foto Depois</p>
                        <input type="file" capture="environment" accept="image/*" className="text-xs w-full" />
                      </div>
                    </div>

                    <div>
                      <label className="text-xs font-semibold block text-slate-700">Nome do Cliente / Responsável no Local</label>
                      <input type="text" className="w-full mt-1 p-2 border rounded-lg text-sm" placeholder="Nome de quem acompanhou e aprova o serviço" value={recebedor} onChange={e => setRecebedor(e.target.value)} />
                    </div>

                    <button onClick={handleFinalizarOS} className="w-full py-3 bg-emerald-600 hover:bg-emerald-700 text-white font-bold rounded-lg shadow transition">
                      ✔ Finalizar e Assinar Relatório Técnico
                    </button>
                  </div>
                ) : (
                  /* TABELA DE OS */
                  <div className="bg-white rounded-xl border border-slate-200 shadow-sm overflow-hidden">
                    <div className="p-4 border-b border-slate-100 flex justify-between items-center">
                      <h2 className="font-bold text-slate-800">Gerenciamento de Ordens de Serviço</h2>
                      <span className="text-xs text-slate-500">{osLista.length} OSs Registradas</span>
                    </div>
                    <div className="overflow-x-auto">
                      <table className="w-full text-left text-sm text-slate-600">
                        <thead className="bg-slate-50 text-xs uppercase font-semibold text-slate-500 border-b">
                          <tr>
                            <th className="p-4">Nº OS</th>
                            <th className="p-4">Cliente / Local</th>
                            <th className="p-4">Equipamento</th>
                            <th className="p-4">Tipo</th>
                            <th className="p-4">Status</th>
                            <th className="p-4 text-right">Ação</th>
                          </tr>
                        </thead>
                        <tbody className="divide-y divide-slate-100">
                          {osLista.map((os) => (
                            <tr key={os.id} className="hover:bg-slate-50">
                              <td className="p-4 font-bold text-slate-800">#{os.codigo_os}</td>
                              <td className="p-4 font-semibold text-slate-800">{os.cliente}<br/><span className="text-xs font-normal text-slate-400">{os.unidade}</span></td>
                              <td className="p-4 text-xs">{os.equipamento}</td>
                              <td className="p-4 text-xs font-semibold uppercase">{os.tipo}</td>
                              <td className="p-4">
                                <span className={`px-2.5 py-1 rounded-full text-[10px] font-bold uppercase ${os.status === 'Concluída' ? 'bg-emerald-100 text-emerald-800' : 'bg-amber-100 text-amber-800'}`}>
                                  {os.status}
                                </span>
                              </td>
                              <td className="p-4 text-right">
                                {os.status !== 'Concluída' && (
                                  <button onClick={() => setOsAtiva(os)} className="px-3 py-1.5 bg-slate-900 hover:bg-slate-800 text-white rounded text-xs font-bold transition">
                                    Atender OS
                                  </button>
                                )}
                              </td>
                            </tr>
                          ))}
                        </tbody>
                      </table>
                    </div>
                  </div>
                )}
              </div>
            )}

            {/* PONTO GPS */}
            {aba === 'ponto' && (
              <div className="max-w-md mx-auto bg-white p-6 rounded-xl border border-slate-200 shadow-sm space-y-6 text-center">
                <div>
                  <h2 className="text-lg font-bold text-slate-800">Cartão de Ponto Georreferenciado</h2>
                  <p className="text-xs text-slate-500">Registro mobile de entrada, intervalo e saída</p>
                </div>

                <div className="p-3 bg-slate-100 rounded-lg text-xs font-mono text-slate-700">
                  📍 {gps ? `Lat: ${gps.lat.toFixed(5)}, Lng: ${gps.lng.toFixed(5)}` : 'Obtendo localização GPS...'}
                </div>

                <div className="grid grid-cols-2 gap-3">
                  <button onClick={() => handleRegistrarPonto('Entrada')} className="p-3 bg-emerald-600 text-white font-bold text-xs rounded-lg shadow hover:bg-emerald-700">🟢 Entrada</button>
                  <button onClick={() => handleRegistrarPonto('Saída Almoço')} className="p-3 bg-amber-500 text-white font-bold text-xs rounded-lg shadow hover:bg-amber-600">🟡 Saída Almoço</button>
                  <button onClick={() => handleRegistrarPonto('Retorno Almoço')} className="p-3 bg-blue-600 text-white font-bold text-xs rounded-lg shadow hover:bg-blue-700">🔵 Retorno Almoço</button>
                  <button onClick={() => handleRegistrarPonto('Saída Jornada')} className="p-3 bg-rose-600 text-white font-bold text-xs rounded-lg shadow hover:bg-rose-700">🔴 Saída Jornada</button>
                </div>

                <div className="border-t pt-4 text-left">
                  <h4 className="text-xs font-bold text-slate-700 mb-2">Marcações de Hoje:</h4>
                  {pontosHoje.length === 0 ? (
                    <p className="text-xs text-slate-400 italic">Nenhum ponto registrado hoje.</p>
                  ) : (
                    <div className="space-y-1.5">
                      {pontosHoje.map((p, idx) => (
                        <div key={idx} className="flex justify-between items-center text-xs p-2 bg-slate-50 rounded border">
                          <span className="font-bold text-slate-700">{p.tipo}</span>
                          <span className="font-mono text-slate-500">{p.hora}</span>
                        </div>
                      ))}
                    </div>
                  )}
                </div>
              </div>
            )}

            {/* ESTOQUE */}
            {aba === 'estoque' && (
              <div className="bg-white rounded-xl border border-slate-200 shadow-sm overflow-hidden">
                <div className="p-4 border-b border-slate-100">
                  <h2 className="font-bold text-slate-800">Controle de Insumos & Materiais de Campo</h2>
                </div>
                <table className="w-full text-left text-sm text-slate-600">
                  <thead className="bg-slate-50 text-xs uppercase font-semibold text-slate-500 border-b">
                    <tr>
                      <th className="p-4">Código</th>
                      <th className="p-4">Material / Insumo</th>
                      <th className="p-4">Saldo em Estoque</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-slate-100">
                    {estoque.map((e) => (
                      <tr key={e.id} className="hover:bg-slate-50">
                        <td className="p-4 font-mono text-xs font-bold">{e.codigo}</td>
                        <td className="p-4 font-semibold text-slate-800">{e.nome}</td>
                        <td className="p-4">
                          <span className={`px-2.5 py-1 rounded-full text-xs font-bold ${e.qtd < 5 ? 'bg-rose-100 text-rose-800' : 'bg-emerald-100 text-emerald-800'}`}>
                            {e.qtd} {e.unidade}
                          </span>
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            )}

            {/* PMOC */}
            {aba === 'pmoc' && (
              <div className="bg-white p-6 rounded-xl border border-slate-200 shadow-sm space-y-4">
                <h2 className="font-bold text-slate-800 text-lg">Plano de Manutenção, Operação e Controle (PMOC)</h2>
                <p className="text-xs text-slate-500">Conformidade com a Lei Federal 13.589/2018 para Sistemas de Climatização.</p>
                <div className="p-4 bg-amber-50 rounded-lg border border-amber-200 text-xs text-amber-800">
                  ⚠️ <strong>Alerta Operacional:</strong> Há 3 rotinas mensais de trocas de filtros G4 com vencimento nesta semana.
                </div>
              </div>
            )}

          </main>

          {/* RODAPÉ */}
          <footer className="bg-slate-900 text-slate-500 text-center py-4 text-xs border-t border-slate-800">
            DAMTECH Engenharia & Serviços © 2026 — Todos os direitos reservados.
          </footer>
        </div>
      );
    }

    ReactDOM.createRoot(document.getElementById('root')).render(<App />);
  </script>
</body>
</html>
