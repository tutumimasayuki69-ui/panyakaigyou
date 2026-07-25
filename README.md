'use client';

import React, { useState, useEffect } from 'react';
import { createClient } from '@supabase/supabase-js';

// Supabaseクライアントの初期化（Vercelの環境変数を自動読み込み）
const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL || '';
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY || '';
const supabase = createClient(supabaseUrl, supabaseAnonKey);

export default function JfcPlanApp() {
  const [user, setUser] = useState<any>(null);
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [loading, setLoading] = useState(false);
  const [step, setStep] = useState(1);

  // フォームデータ
  const [formData, setFormData] = useState({
    industry: '',
    motivation: '',
    experience: '',
    equipmentCost: 500,
    workingCapital: 300,
    ownCapital: 300,
    loanAmount: 500,
    sales: 150,
    costOfGoods: 45,
    personnelCost: 30,
    rent: 15,
    otherExpense: 10,
    repaymentMonths: 84,
  });

  // ログイン状態の確認
  useEffect(() => {
    const checkUser = async () => {
      const { data: { session } } = await supabase.auth.getSession();
      setUser(session?.user ?? null);
    };
    checkUser();

    const { data: { subscription } } = supabase.auth.onAuthStateChange((_event, session) => {
      setUser(session?.user ?? null);
    });

    return () => subscription.unsubscribe();
  }, []);

  // 新規登録・ログイン処理
  const handleAuth = async (type: 'login' | 'signup') => {
    setLoading(true);
    let error;
    if (type === 'signup') {
      const { error: err } = await supabase.auth.signUp({ email, password });
      error = err;
    } else {
      const { error: err } = await supabase.auth.signInWithPassword({ email, password });
      error = err;
    }
    if (error) alert(error.message);
    else alert(type === 'signup' ? '確認メールを送信しました！' : 'ログインしました！');
    setLoading(false);
  };

  // ログアウト処理
  const handleLogout = () => supabase.auth.signOut();

  // Supabaseへデータを保存
  const saveData = async () => {
    if (!user) {
      alert('保存するにはログインが必要です！');
      return;
    }
    setLoading(true);
    const { error } = await supabase.from('business_plans').insert([
      {
        user_id: user.id,
        industry: formData.industry,
        motivation: formData.motivation,
        experience: formData.experience,
        equipment_cost: formData.equipmentCost,
        working_capital: formData.workingCapital,
        own_capital: formData.ownCapital,
        loan_amount: formData.loanAmount,
        sales: formData.sales,
        cost_of_goods: formData.costOfGoods,
        personnel_cost: formData.personnelCost,
        rent: formData.rent,
        other_expense: formData.otherExpense,
        repayment_months: formData.repaymentMonths,
      },
    ]);

    if (error) alert('保存に失敗しました: ' + error.message);
    else alert('クラウド（Supabase）に正常に保存されました！');
    setLoading(false);
  };

  // 収支計算 logic
  const grossProfit = formData.sales - formData.costOfGoods;
  const totalExpenses = formData.personnelCost + formData.rent + formData.otherExpense;
  const operatingProfit = grossProfit - totalExpenses;
  const monthlyRepayment = formData.loanAmount > 0 ? Math.round(formData.loanAmount / (formData.repaymentMonths / 12) / 12) : 0;
  const netCashFlow = operatingProfit - monthlyRepayment;

  return (
    <div style={{ maxWidth: '800px', margin: '0 auto', padding: '20px', fontFamily: 'sans-serif' }}>
      <header style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', borderBottom: '2px solid #eee', pb: '10px', mb: '20px' }}>
        <h1 style={{ fontSize: '20px', fontWeight: 'bold' }}>日本政策金融公庫 創業計画書作成ツール</h1>
        <div>
          {user ? (
            <div style={{ display: 'flex', gap: '10px', alignItems: 'center' }}>
              <span style={{ fontSize: '12px' }}>👤 {user.email}</span>
              <button onClick={handleLogout} style={{ padding: '4px 8px', cursor: 'pointer' }}>ログアウト</button>
            </div>
          ) : (
            <span style={{ fontSize: '12px', color: '#666' }}>未ログイン（保存機能を使うにはログインが必要です）</span>
          )}
        </div>
      </header>

      {/* ログインフォーム（未ログイン時のみ簡易表示） */}
      {!user && (
        <div style={{ background: '#f9f9f9', padding: '15px', borderRadius: '8px', marginBottom: '20px' }}>
          <h3>ログイン / アカウント作成</h3>
          <div style={{ display: 'flex', gap: '10px', marginTop: '10px' }}>
            <input
              type="email"
              placeholder="メールアドレス"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              style={{ padding: '8px', flex: 1 }}
            />
            <input
              type="password"
              placeholder="パスワード"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              style={{ padding: '8px', flex: 1 }}
            />
            <button onClick={() => handleAuth('login')} disabled={loading} style={{ padding: '8px 15px', cursor: 'pointer' }}>ログイン</button>
            <button onClick={() => handleAuth('signup')} disabled={loading} style={{ padding: '8px 15px', cursor: 'pointer' }}>新規登録</button>
          </div>
        </div>
      )}

      {/* ステップ切替ナビ */}
      <div style={{ display: 'flex', gap: '10px', marginBottom: '20px' }}>
        <button onClick={() => setStep(1)} style={{ padding: '10px', background: step === 1 ? '#0070f3' : '#eee', color: step === 1 ? '#fff' : '#000', border: 'none', borderRadius: '5px', cursor: 'pointer' }}>1. 動機・経験</button>
        <button onClick={() => setStep(2)} style={{ padding: '10px', background: step === 2 ? '#0070f3' : '#eee', color: step === 2 ? '#fff' : '#000', border: 'none', borderRadius: '5px', cursor: 'pointer' }}>2. 資金・調達計画</button>
        <button onClick={() => setStep(3)} style={{ padding: '10px', background: step === 3 ? '#0070f3' : '#eee', color: step === 3 ? '#fff' : '#000', border: 'none', borderRadius: '5px', cursor: 'pointer' }}>3. 収支・返済シミュレーション</button>
      </div>

      {/* Step 1 */}
      {step === 1 && (
        <div>
          <h2>Step 1: 動機・経験</h2>
          <div style={{ marginBottom: '15px' }}>
            <label style={{ display: 'block', marginBottom: '5px' }}>業種:</label>
            <input type="text" value={formData.industry} onChange={(e) => setFormData({ ...formData, industry: e.target.value })} style={{ width: '100%', padding: '8px' }} placeholder="例：カフェ・飲食店" />
          </div>
          <div style={{ marginBottom: '15px' }}>
            <label style={{ display: 'block', marginBottom: '5px' }}>創業の動機:</label>
            <textarea value={formData.motivation} onChange={(e) => setFormData({ ...formData, motivation: e.target.value })} style={{ width: '100%', height: '80px', padding: '8px' }} placeholder="なぜこの事業を始めるのか..." />
          </div>
          <div style={{ marginBottom: '15px' }}>
            <label style={{ display: 'block', marginBottom: '5px' }}>略歴・過去の経験:</label>
            <textarea value={formData.experience} onChange={(e) => setFormData({ ...formData, experience: e.target.value })} style={{ width: '100%', height: '80px', padding: '8px' }} placeholder="これまでの職歴や業界経験..." />
          </div>
        </div>
      )}

      {/* Step 2 */}
      {step === 2 && (
        <div>
          <h2>Step 2: 必要な資金と調達方法（単位：万円）</h2>
          <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '15px' }}>
            <div>
              <h3>必要な資金</h3>
              <label>設備資金: <input type="number" value={formData.equipmentCost} onChange={(e) => setFormData({ ...formData, equipmentCost: Number(e.target.value) })} style={{ width: '100%', padding: '8px' }} /></label>
              <label style={{ marginTop: '10px', display: 'block' }}>運転資金: <input type="number" value={formData.workingCapital} onChange={(e) => setFormData({ ...formData, workingCapital: Number(e.target.value) })} style={{ width: '100%', padding: '8px' }} /></label>
              <p><strong>必要資金 合計: {formData.equipmentCost + formData.workingCapital} 万円</strong></p>
            </div>
            <div>
              <h3>調達方法</h3>
              <label>自己資金: <input type="number" value={formData.ownCapital} onChange={(e) => setFormData({ ...formData, ownCapital: Number(e.target.value) })} style={{ width: '100%', padding: '8px' }} /></label>
              <label style={{ marginTop: '10px', display: 'block' }}>日本公庫からの融資希望額: <input type="number" value={formData.loanAmount} onChange={(e) => setFormData({ ...formData, loanAmount: Number(e.target.value) })} style={{ width: '100%', padding: '8px' }} /></label>
              <p><strong>調達合計: {formData.ownCapital + formData.loanAmount} 万円</strong></p>
            </div>
          </div>
        </div>
      )}

      {/* Step 3 */}
      {step === 3 && (
        <div>
          <h2>Step 3: 収支計画・返済シミュレーション（月額：万円）</h2>
          <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '15px' }}>
            <div>
              <h3>売上・原価・経費入力</h3>
              <label>売上高: <input type="number" value={formData.sales} onChange={(e) => setFormData({ ...formData, sales: Number(e.target.value) })} style={{ width: '100%', padding: '8px' }} /></label>
              <label style={{ marginTop: '8px', display: 'block' }}>売上原価: <input type="number" value={formData.costOfGoods} onChange={(e) => setFormData({ ...formData, costOfGoods: Number(e.target.value) })} style={{ width: '100%', padding: '8px' }} /></label>
              <label style={{ marginTop: '8px', display: 'block' }}>人件費: <input type="number" value={formData.personnelCost} onChange={(e) => setFormData({ ...formData, personnelCost: Number(e.target.value) })} style={{ width: '100%', padding: '8px' }} /></label>
              <label style={{ marginTop: '8px', display: 'block' }}>家賃: <input type="number" value={formData.rent} onChange={(e) => setFormData({ ...formData, rent: Number(e.target.value) })} style={{ width: '100%', padding: '8px' }} /></label>
              <label style={{ marginTop: '8px', display: 'block' }}>その他経費: <input type="number" value={formData.otherExpense} onChange={(e) => setFormData({ ...formData, otherExpense: Number(e.target.value) })} style={{ width: '100%', padding: '8px' }} /></label>
            </div>
            <div style={{ background: '#f5f5f5', padding: '15px', borderRadius: '8px' }}>
              <h3>📊 月間シミュレーション結果</h3>
              <p>売上総利益（粗利）: <strong>{grossProfit} 万円</strong></p>
              <p>経費合計: <strong>{totalExpenses} 万円</strong></p>
              <p style={{ color: operatingProfit >= 0 ? 'green' : 'red' }}>営業利益: <strong>{operatingProfit} 万円</strong></p>
              <hr />
              <p>推定月額返済額（元金のみ推定）: <strong>約 {monthlyRepayment} 万円</strong></p>
              <p style={{ fontSize: '18px', fontWeight: 'bold', color: netCashFlow >= 0 ? 'blue' : 'red' }}>
                最終手元残り資金: {netCashFlow} 万円 /月
              </p>
            </div>
          </div>
        </div>
      )}

      {/* クラウド保存ボタン */}
      <div style={{ marginTop: '30px', borderTop: '2px solid #eee', paddingTop: '20px', textAlign: 'center' }}>
        <button
          onClick={saveData}
          disabled={loading}
          style={{ padding: '12px 24px', background: '#0070f3', color: '#fff', border: 'none', borderRadius: '6px', fontSize: '16px', fontWeight: 'bold', cursor: 'pointer' }}
        >
          ☁️ Supabaseに創業計画を保存する
        </button>
      </div>
    </div>
  );
}
