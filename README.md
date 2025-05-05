// FishShop.jsx
import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { ShoppingCart, Phone, MapPin, Trash2, BarChart3 } from "lucide-react";
import { Dialog, DialogContent, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { Input } from "@/components/ui/input";
import emailjs from "emailjs-com";

const products = [
  {
    id: 1,
    name: "Cá La Hán",
    price: 250000,
    image: "https://source.unsplash.com/400x300/?flowerhorn-fish",
    description: "Cá La Hán khoẻ mạnh, màu sắc đẹp."
  },
  {
    id: 2,
    name: "Cá 7 Màu",
    price: 50000,
    image: "https://source.unsplash.com/400x300/?guppy-fish",
    description: "Bầy cá 7 màu sinh sản tốt, đủ màu."
  },
  {
    id: 3,
    name: "Thuốc trị nấm cá",
    price: 80000,
    image: "https://source.unsplash.com/400x300/?fish-medicine",
    description: "Hiệu quả nhanh chóng, an toàn."
  }
];

export default function FishShop() {
  const [cart, setCart] = useState([]);
  const [checkoutOpen, setCheckoutOpen] = useState(false);
  const [adminOpen, setAdminOpen] = useState(false);
  const [orders, setOrders] = useState(() => JSON.parse(localStorage.getItem("fishshop-orders")) || []);
  const [buyerInfo, setBuyerInfo] = useState({ name: "", phone: "", address: "" });

  useEffect(() => {
    const storedCart = localStorage.getItem("fishshop-cart");
    if (storedCart) setCart(JSON.parse(storedCart));
  }, []);

  useEffect(() => {
    localStorage.setItem("fishshop-cart", JSON.stringify(cart));
  }, [cart]);

  useEffect(() => {
    localStorage.setItem("fishshop-orders", JSON.stringify(orders));
  }, [orders]);

  const addToCart = (product) => {
    setCart((prev) => {
      const exists = prev.find((item) => item.id === product.id);
      if (exists) {
        return prev.map((item) =>
          item.id === product.id ? { ...item, qty: item.qty + 1 } : item
        );
      }
      return [...prev, { ...product, qty: 1 }];
    });
  };

  const removeFromCart = (id) => {
    setCart((prev) => prev.filter((item) => item.id !== id));
  };

  const total = cart.reduce((sum, item) => sum + item.price * item.qty, 0);

  const handleOrder = () => {
    const orderData = {
      id: Date.now(),
      name: buyerInfo.name,
      phone: buyerInfo.phone,
      address: buyerInfo.address,
      items: cart,
      total
    };

    const templateParams = {
      name: orderData.name,
      phone: orderData.phone,
      address: orderData.address,
      items: orderData.items.map(i => `${i.name} x${i.qty}`).join(", "),
      total: orderData.total.toLocaleString() + " đ",
      email_to: "jasminiinguyen@gmail.com"
    };

    emailjs.send("service_id", "template_id", templateParams, "public_key")
      .then(() => {
        alert("Đặt hàng thành công! Shop sẽ liên hệ với bạn sớm.");
        setOrders((prev) => [...prev, orderData]);
        setCart([]);
        localStorage.removeItem("fishshop-cart");
        setCheckoutOpen(false);
      })
      .catch((err) => {
        console.error("Lỗi gửi email:", err);
        alert("Gửi thất bại. Vui lòng thử lại!");
      });
  };

  return (
    <div className="p-6 max-w-6xl mx-auto">
      <header className="text-center mb-10">
        <h1 className="text-4xl font-bold">Cá Cảnh Kuro</h1>
        <p className="text-gray-600">Shop bán cá cảnh, hồ cá & phụ kiện</p>
        <Button
          variant="outline"
          className="mt-4"
          onClick={() => setAdminOpen(true)}
        >
          <BarChart3 className="w-4 h-4 mr-2" /> Xem thống kê đơn hàng
        </Button>
      </header>

      <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
        {products.map((product) => (
          <Card key={product.id} className="rounded-2xl shadow-md">
            <img
              src={product.image}
              alt={product.name}
              className="w-full h-48 object-cover rounded-t-2xl"
            />
            <CardContent className="p-4">
              <h2 className="text-xl font-semibold mb-2">{product.name}</h2>
              <p className="text-gray-700 mb-2">{product.description}</p>
              <p className="font-bold text-lg text-green-600 mb-4">
                {product.price.toLocaleString()} đ
              </p>
              <Button className="w-full" onClick={() => addToCart(product)}>
                <ShoppingCart className="w-4 h-4 mr-2" /> Thêm vào giỏ
              </Button>
            </CardContent>
          </Card>
        ))}
      </div>

      <div className="mt-12">
        <h2 className="text-2xl font-bold mb-4">🛒 Giỏ hàng của bạn</h2>
        {cart.length === 0 ? (
          <p className="text-gray-600">Chưa có sản phẩm nào trong giỏ hàng.</p>
        ) : (
          <div className="space-y-4">
            {cart.map((item) => (
              <div
                key={item.id}
                className="flex justify-between items-center border p-4 rounded-lg"
              >
                <div>
                  <p className="font-semibold">{item.name}</p>
                  <p className="text-sm text-gray-600">
                    {item.qty} x {item.price.toLocaleString()} đ
                  </p>
                </div>
                <div className="flex items-center gap-4">
                  <p className="font-bold text-green-600">
                    {(item.qty * item.price).toLocaleString()} đ
                  </p>
                  <Button
                    variant="outline"
                    size="icon"
                    onClick={() => removeFromCart(item.id)}
                  >
                    <Trash2 className="w-4 h-4" />
                  </Button>
                </div>
              </div>
            ))}
            <div className="text-right font-bold text-xl text-blue-600">
              Tổng cộng: {total.toLocaleString()} đ
            </div>
            <Button
              className="mt-4 w-full bg-green-600 text-white hover:bg-green-700"
              onClick={() => setCheckoutOpen(true)}
            >
              Tiến hành thanh toán
            </Button>
          </div>
        )}
      </div>

      <Dialog open={checkoutOpen} onOpenChange={setCheckoutOpen}>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>Thông tin đặt hàng</DialogTitle>
          </DialogHeader>
          <div className="space-y-4">
            <Input
              placeholder="Họ tên người nhận"
              value={buyerInfo.name}
              onChange={(e) => setBuyerInfo({ ...buyerInfo, name: e.target.value })}
            />
            <Input
              placeholder="Số điện thoại"
              value={buyerInfo.phone}
              onChange={(e) => setBuyerInfo({ ...buyerInfo, phone: e.target.value })}
            />
            <Input
              placeholder="Địa chỉ giao hàng"
              value={buyerInfo.address}
              onChange={(e) => setBuyerInfo({ ...buyerInfo, address: e.target.value })}
            />
            <Button onClick={handleOrder} className="w-full bg-blue-600 text-white hover:bg-blue-700">
              Xác nhận đặt hàng
            </Button>
          </div>
        </DialogContent>
      </Dialog>

      <Dialog open={adminOpen} onOpenChange={setAdminOpen}>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>📊 Đơn hàng đã nhận</DialogTitle>
          </DialogHeader>
          <div className="space-y-4 max-h-[60vh] overflow-y-auto">
            {orders.length === 0 ? (
              <p className="text-gray-600">Chưa có đơn hàng nào.</p>
            ) : (
              orders.map((order) => (
                <div key={order.id} className="border p-4 rounded-lg">
                  <p><strong>Khách:</strong> {order.name} - {order.phone}</p>
                  <p><strong>Địa chỉ:</strong> {order.address}</p>
                  <p><strong>Sản phẩm:</strong> {order.items.map(i => `${i.name} x${i.qty}`).join(", ")}</p>
                  <p className="text-right font-semibold text-green-700">
                    Tổng: {order.total.toLocaleString()} đ
                  </p>
                </div>
              ))
            )}
          </div>
        </DialogContent>
      </Dialog>

      <footer className="mt-12 text-center text-gray-600">
        <div className="flex flex-col items-center gap-2">
          <div className="flex items-center gap-2">
            <Phone className="w-4 h-4" /> <span>0909 123 456</span>
          </div>
          <div className="flex items-center gap-2">
            <MapPin className="w-4 h-4" /> <span>123 Trần Hưng Đạo, Cần Thơ</span>
          </div>
        </div>
        <p className="mt-4">© 2025 Cá Cảnh Kuro. All rights reserved.</p>
      </footer>
    </div>
  );
}
