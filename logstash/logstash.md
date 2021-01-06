
#### Logstash Filter Plugin Örnekleri
* **Drop:** İf koşulu ile birlikte kullanılır. Bu şart sağlandığında bu document eklenmez. Örneğin; Ödeme Türü Sodexo olanlar eklenmez.
  
        filter {
            json {
                source => "message"
            }
            if([orderPayment] == "Sodexo"){
                drop {}
            }
        }
* **Mutate:** Alanlar üzerinde genel değişiklikler yapmamıza olanak tanır. Alanları yeniden adlandırabilir, değiştirebilir veya kaldırabiliriz. 
  
  * Rename:


        filter {
            mutate {
                rename => { "orderDate" => "timestamp" }
            }

   * Update: Alan yok ise hiç bir işlem yapılmaz.


            filter {
                mutate {
                    update => { "orderPaymentType" => "Multinet" }
                }
            }

   * **Coerce:** Boş bir alana varsayılan bir değer verir.


            filter {
                mutate {
                    coerce => { "field1" => "default_value" }
                }
            }
                
   * **Add_Field: **


            filter {
                mutate {
                    add_field => { "orderedEmail}" => "cihattpskl@gamil.com" }
                }
            }
                
    * **Remove_Field:**


            filter {
                mutate {
                    remove_field => [ "message","host","@timestamp","@version"]
                }
            }